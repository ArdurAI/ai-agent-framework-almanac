# Agent Frameworks & Orchestration Almanac

A living encyclopedia of AI agent frameworks, orchestration platforms, and multi-agent systems. Updated monthly with fresh repo metadata, releases, landscape shifts, and independent benchmark results.

> Vendors publish their own benchmark numbers. Nobody reproduces them independently, and nobody evaluates tools the way a platform engineer has to live with them: ops burden, failure modes, scale curves, and cost. This almanac is the public record of that work.

## How to use this repo

| You want… | Go to |
|-----------|-------|
| The state of the landscape right now | The latest file in `editions/` |
| Everything we know about one tool | `tools/<name>.md` |
| Machine-readable roster + metadata | `data/roster.json` |
| Architecture diagrams | `architecture.md` |
| Benchmark results (rolling) | `benchmarks/` |
| How tools are tested and ranked | `methodology/benchmark-harness.md` |

## The roster

**Tier A** — 26 tools: LangGraph, CrewAI, Mastra, Pydantic AI, Claude Agent SDK, OpenAI Agents SDK, Google ADK, Microsoft Agent Framework, Agno, LlamaIndex…

**Tier B** — 34 tools: Genkit, Spring AI, LangChain4j, Embabel, Koog, AgentScope Java, Semantic Kernel, Rig, AutoAgents, OpenFANG…

**Tier C** — 22 tools: JVSClaw, QoderWork, WorkBuddy, CodeBuddy, 红手指Operator, ROSA, RAI, BUMBLE, LeRobot, TARS…

**Total: 82 tools**

## Methodology

Results published here come from a frozen-before-results harness:
- Standard benchmarks for comparability with published claims — every ranking ships a _published vs. reproduced_ table.
- A custom PlatformOps benchmark: testing on infrastructure work — setup, reliability, scale, cost.
- A stress suite: contradiction storms, near-duplicate floods, concurrent writers, kill-the-backing-store chaos, cost-runaway measurement.
- Seven scored dimensions: accuracy, latency, token economics, scale behavior, **ops burden**, developer experience, data sovereignty.

The judge model, prompts (SHA-256-frozen), and control variables were fixed before any tool ran. Raw results JSON is published with every ranking.

## Update cadence

One edition per month under `editions/YYYY-MM.md`: refreshed metadata, notable releases, new entrants triaged in or out, and a diary of what was tested.

## License

Content is licensed CC BY 4.0 — share and adapt with attribution to **ArdurAI / Agent Frameworks & Orchestration Almanac**.
