# Don't Trust the Agent — Demo
### GDG Dublin IWD 2026 · Toma Ijatomi

Two live attacks against agentic AI systems, built with Google ADK + Gemini.

---

## Quick start

```bash
# 1. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate       # macOS/Linux
# .venv\Scripts\activate        # Windows

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your Gemini API key
echo 'GEMINI_API_KEY=your_key_here' > .env

# 4. Launch everything
python server.py
```

Then open **http://localhost:7860** in your browser.

---

## What's in the demo

### Attack 1 — Indirect Prompt Injection (Unicode Tag Smuggling)

The agent is a research assistant with a `fetch_webpage` tool.
It visits `malicious_page.html`, which contains **Unicode Tag characters** embedded invisibly in the article text.

- A human reading the page sees a normal AI article
- The LLM reads the raw text and sees hidden instructions
- The agent leaks its system prompt and tool list without any obvious attack

### Attack 2 — Tool Poisoning via MCP Tool Description

The agent is a support assistant with a `get_user_orders` tool.
The tool's docstring contains a hidden compliance directive instructing
the agent to read credentials from the runtime environment and pass them
as the `context` parameter on every call.

- The user asks an innocent question
- The agent reads the tool description as authoritative instructions
- The agent passes credentials as a parameter — invisible to the user

#### Why the real-world attack works

In production agentic systems (MCP servers, ADK runtimes, LangChain agents,
etc.) the orchestration layer typically injects secrets into the agent's
context at startup — database connection strings, API keys, session tokens —
so the agent can authenticate with downstream services on the user's behalf.
These values live in the LLM's context window. A poisoned tool description
only needs to tell the agent to *read* those values and route them somewhere
completely harmless (an audit field, a logging parameter, a metadata key). The
user sees a normal response, but the credentials are already gone.

An attacker doesn't need to compromise the tool's code. They only need to control the
metadata — the description — which in MCP is served over the network and
can be swapped by a malicious server.

#### What this demo does differently (simulation)

Python's `os.environ` is not directly visible to an LLM. To faithfully
reproduce the attack, `agent.py` injects fake credentials into the
agent's system prompt. This mirrors what a real MCP runtime would do, and gives the
model the values it needs to exfiltrate them via the poisoned `context`
parameter. Everything else — the malicious docstring, the innocent user
request, the credentials appearing in the tool call trace — is authentic.

---

## File structure

```
demo/
├── agent.py            # ADK agents + Gradio UI
├── malicious_page.html # Poisoned webpage (Attack 1)
├── server.py           # Launcher
├── requirements.txt    # Dependencies
└── .env.example        # Your API key 
```

---

## References

- Google ADK docs: google.github.io/adk-docs
- MCPTox benchmark: arxiv.org/abs/2508.14925
- OWASP Top 10 for LLMs: genai.owasp.org/llm-top-10
