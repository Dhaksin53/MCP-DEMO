# MCP demo

A small demo that wires **[Model Context Protocol (MCP)](https://modelcontextprotocol.io/)** tools into a **[LangGraph](https://langchain-ai.github.io/langgraph/)** ReAct agent powered by **[Groq](https://groq.com/)**. It shows two transport styles in one client: **stdio** (local math tools) and **streamable HTTP** (a mock weather tool).

## What it does

- **Math MCP server** (`mathserver.py`): exposes `add` and `multiple` (multiply) as MCP tools over **stdio**. The client spawns this process automatically.
- **Weather MCP server** (`weather.py`): exposes a stub `get_weather` tool over **streamable HTTP**. You start this server yourself; the client calls `http://localhost:8000/mcp`.
- **Agent client** (`client.py`): uses `MultiServerMCPClient` to collect tools from both servers, then runs a ReAct agent (`create_react_agent`) with `ChatGroq` (`llama-3.1-8b-instant`). It runs two example prompts: a math question and a weather question.

## Requirements

- **Python 3.13+** (see `.python-version` / `pyproject.toml`)
- A **Groq API key** ([Groq Console](https://console.groq.com/))

## Project layout

| File | Role |
|------|------|
| `client.py` | LangGraph + Groq agent; connects to both MCP servers |
| `mathserver.py` | FastMCP server — stdio — `add`, `multiple` |
| `weather.py` | FastMCP server — streamable HTTP — `get_weather` (mock) |
| `main.py` | Placeholder entrypoint (`Hello from mcp-demo!`) |
| `pyproject.toml` | Project metadata and dependencies (uv/pip compatible) |
| `requirements.txt` | Pip-style dependency list |

## Setup

### 1. Clone and create a virtual environment

```bash
cd mcp-demo
python -m venv .venv
```

**Windows (PowerShell):**

```powershell
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
source .venv/bin/activate
```

### 2. Install dependencies

Using **pip**:

```bash
pip install -r requirements.txt
pip install python-dotenv
```

Or using **uv** (if you use `uv.lock`):

```bash
uv sync
uv pip install python-dotenv
```

> `client.py` loads environment variables with `python-dotenv`; install it if it is not already pulled in by your environment.

### 3. Environment variables

Create a `.env` file in the project root (this file is gitignored):

```env
GROQ_API_KEY=your_groq_api_key_here
```

Never commit real API keys. For collaborators, document variables in `.env.example` if you add one.

## How to run

You need **two terminals** (or run the weather server in the background).

### Terminal 1 — Weather MCP (HTTP)

From the **project root** (so imports and paths stay consistent):

```bash
python weather.py
```

Leave this running. The client expects the MCP endpoint at **`http://localhost:8000/mcp`**.

### Terminal 2 — Agent client

Activate the same venv, stay in the **project root**, then:

```bash
python client.py
```

The client will:

1. Spawn `python mathserver.py` as the stdio math server (ensure `mathserver.py` is on the PATH or adjust the `args` in `client.py` to an absolute path if needed).
2. Connect to the weather server over HTTP.
3. Print the agent’s answers for the built-in math and weather prompts.

## Configuration notes

- **Math server path**: In `client.py`, the math server is started with `args: ["mathserver.py"]`. That works when your current working directory is the repo root. If you run `client.py` from elsewhere, set `args` to the **absolute path** of `mathserver.py`.
- **Weather URL**: If you change host, port, or path in `weather.py`, update the `"url"` under the `weather` entry in `MultiServerMCPClient` in `client.py`.
- **Model**: The Groq model is set in `client.py` (`ChatGroq(model="llama-3.1-8b-instant")`). Change it if you prefer another Groq-supported model.

## Dependencies (high level)

- `mcp` — MCP SDK / FastMCP servers
- `langchain-mcp-adapters` — `MultiServerMCPClient` and tool bridging
- `langgraph` — `create_react_agent`
- `langchain-groq` — Groq chat model
- `python-dotenv` — load `.env` in `client.py`

## Extending the demo

- Add real weather logic inside `get_weather` in `weather.py` (HTTP API, etc.).
- Register more tools on either server; the agent will see them after `get_tools()`.
- Replace the hard-coded prompts in `client.py` with CLI args or a small REPL.

## License

Specify your license here (e.g. MIT), or remove this section if the repo is private and unlicensed.
