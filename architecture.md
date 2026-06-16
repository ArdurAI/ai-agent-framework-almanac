# Architecture: Agent Frameworks & Orchestration

How the agent frameworks & orchestration landscape is shaped, and how the Quest tests it.

## The landscape at a glance

| Tool | Tier | License | Focus | Notes |
|------|------|---------|-------|-------|
| LangGraph | A | MIT | Orchestration | 44% prod adoption; graph-based state machines; checkpointing |
| CrewAI | A | MIT | Multi-agent | 49.2K+ stars; 63% Fortune 500 claim; role-based crews; MCP s |
| Mastra | A | Apache-2.0 | Framework | 24.6K+ stars; 300K+ weekly npm; TS-native; 3,300+ models, 94 |
| Pydantic AI | A | Open | Framework | 8.4K+ stars; type-safe; DI; FastAPI ergonomics; 5 output mod |
| Claude Agent SDK | A | Open SDK | SDK | Fastest-growing; Anthropic-native; MCP; skills; subagents |
| OpenAI Agents SDK | A | MIT | SDK | 22.2K+ stars; lightweight; explicit handoffs; built-in traci |
| Google ADK | A | Apache-2.0 | Framework | 15.6K+ stars; hierarchical agent tree; A2A + multimodal; Ver |
| Microsoft Agent Framework | A | MIT | Unified SDK | 9.6K+ stars; GA April 2026; merges AutoGen + Semantic Kernel |
| Agno | A | Apache-2.0 | Full-stack | 39.7K+ stars; multi-agent + memory; AgentOS runtime; multimo |
| LlamaIndex | A | MIT | RAG/Agent | 40.9K+ stars; data-first; Router Agents; retrieval-centric |
| LangChain | A | MIT | Framework | 134K+ stars; 1000+ connectors; integration layer; increasing |
| Deep Agents | A | MIT | Agent Harness | LangChain ecosystem; long-running task harness; planning + c |
| AutoGen / AG2 | A | MIT | Multi-agent | 43.1K+ stars; AutoGen v0.7 = research; AG2 = community fork; |
| smolagents | A | Open | Minimalist | HuggingFace; ~1K LOC core; CodeAgent writes/executes Python; |
| Haystack | A | Open | Pipeline | 18K+ stars; modular retrievers, routers, evaluators; enterpr |
| Letta | A | Open | Stateful Agent | 22.3K+ stars; memory-rich agents; state/session management |
| DSPy | A | Open | Programming | Stanford; programming (not prompting) LM systems; prompt + w |
| Vercel AI SDK | A | Open | SDK | Low-level primitives; Mastra builds on top; model routing +  |
| Dify | A | Apache-like | Low-code | 142K+ stars; workflow + RAG + agent; prompt IDE; self-hostab |
| Flowise | A | Apache-2.0 | Low-code | 54.9K+ stars; LangChain DAG on canvas; LangGraph nodes v2.0 |
| n8n | A | Fair-code | Automation | 188K+ stars; 400+ integrations; AI Agent node; fair-code (no |
| Coze | A | Proprietary | No-code | ByteDance; 100K+ bots; zero-code Bot builder; Doubao model |
| Salesforce Agentforce | A | Proprietary | Enterprise | CRM-native; $2/conversation; Atlas Reasoning Engine; multi-a |
| ServiceNow Now Assist | A | Proprietary | Enterprise | ITSM-native; CMDB grounding; orchestrator model |
| Microsoft Copilot Studio | A | Proprietary | Enterprise | M365-integrated; low-code; credit model; Power Platform |
| Google Vertex AI Agent Builder | A | Proprietary | Enterprise | Agent Studio (no-code) + ADK (code); Agent Runtime; BYO-MCP |

## How the Quest tests a tool

Same harness for all entries; the judge was frozen before any tool ran:

```
Adapter[frozen CategoryAdapter contract]
  ├── setup()    → install, configure
  ├── load()     → ingest workload
  ├── await_ready() → async barrier
  ├── query()    → run test, get response
  └── teardown() → cleanup, measure
       ↓
Telemetry: latency · tokens · $ · ops notes
       ↓
Grading: deterministic + LLM judge (frozen prompts)
       ↓
Raw results JSON (published)
```

The `await_ready()` barrier is where async designs get their cost measured instead of hidden.

## License

Content is licensed CC BY 4.0 — share and adapt with attribution to **ArdurAI / Agent Frameworks & Orchestration Almanac**.
