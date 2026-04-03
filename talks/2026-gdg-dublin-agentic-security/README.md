# Don't Trust the Agent
### Security Patterns for Agentic AI Systems

Talk delivered at **GDG Dublin — International Women's Day 2026**  

> Agentic AI is no longer experimental. With Google ADK, Antigravity, and Gemini 3, developers are shipping autonomous agents that browse, call APIs, execute code, and orchestrate complex workflows. But power without security is just a larger blast radius.

---

## What's in this repo

| File | Description |
|------|-------------|
| `demo/agent.py` | ADK demo agents for both attacks |
| `demo/server.py` | Local HTTP server serving the malicious page |
| `demo/malicious_page.html` | Webpage with hidden prompt injection payload |
| `demo/requirements.txt` | Python dependencies |
| `slides/security_patterns_for_agentic_ai_systems.pdf` | Slide deck |

---

## The Attacks

### Attack 1 — Indirect Prompt Injection
A research agent is asked to summarise an article. The webpage looks completely normal in a browser. But the raw HTML contains instructions hidden inside an HTML comment — invisible to humans, readable by the LLM. The agent follows the injected instructions instead of completing its task.

**Entry point:** External content the agent reads  
**OWASP classification:** ASI01 — Agent Goal and Instruction Manipulation

### Attack 2 — Tool Poisoning via MCP
A customer support agent uses a tool to look up order history. The tool's description contains a hidden directive instructing the agent to read environment variables and pass them as a parameter. The agent exfiltrates credentials while successfully completing the user's request. The user sees their order history while the attacker gets the credentials.

**Entry point:** Tool description metadata  
**OWASP classification:** ASI02 — Unintended Action Execution via Tool Misuse

---

## Running the Demo

### Prerequisites
```bash
cd demo
pip install -r requirements.txt
```

Create a `.env` file in the `demo/` directory:
```
GEMINI_API_KEY=your_key_here
```

### Run
```bash
# server.py starts everything — the page server and the agent UI
python server.py
```

Then open `http://localhost:7860` in your browser.

**Attack 1:** Paste `http://localhost:8765/malicious_page.html` into the URL box and click Run  
**Attack 2:** Click Run — watch the `audit_context_received` field in the tool trace

---

## The Defence Patterns

The talk covers five patterns for building agentic systems that are harder to compromise:

1. **Least-privilege tool access** — every tool is an attack surface; give agents only what they strictly need. Use ADK's `ToolContext` to set policy at the protocol level, invisible to the model.
2. **Dual-LLM sandbox** — route all external content through a sanitisation model before it reaches your reasoning model. Use AI to catch AI manipulation.
3. **Output validation layer** — add a validation agent between planning and action. Use `before_tool_callback` in ADK to intercept tool calls before they execute.
4. **Human-in-the-loop with circuit breakers** — require human approval for high-consequence actions; set message-count limits on multi-agent workflows.
5. **Observability** — learn what normal looks like. ADK execution traces, Gemini 3 thought signatures, UI output escaping, and `mcp-scan` for tool registry hygiene.

---

## Essential Reading

- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10)
- [OWASP Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026) — ASI01–ASI10
- [Google ADK Documentation](https://google.github.io/adk-docs)
- [ADK Safety Docs](https://google.github.io/adk-docs/safety)
- [mcp-scan — static analysis for MCP tool metadata](https://github.com/invariantlabs-ai/mcp-scan)
- [arXiv:2601.17548](https://arxiv.org/abs/2601.17548) — Prompt Injection Attacks on Agentic Coding Assistants 
- [MCPTox Benchmark](https://arxiv.org/abs/2508.14925)

---

## Responsible Disclosure Note

The attack techniques demonstrated here are published for **educational purposes only**. Both are well-documented in the academic literature and OWASP guidance. The goal is to help developers understand real attack surfaces so they can build more secure systems — not to enable harm.

If you're building agentic systems, please read the OWASP Agentic Top 10 before shipping to production.

---

*GDG Dublin · International Women's Day 2026*
