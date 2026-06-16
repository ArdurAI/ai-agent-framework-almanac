# Project Intent & Philosophy

## Why this almanac exists

The AI agent framework landscape is exploding. Every week, a new "must-have" orchestration layer launches, a blog post claims 10x multi-agent performance gains, and a vendor announces the next revolution in autonomous reasoning. But **nobody independently verifies these claims**. The benchmarks are self-reported, the comparisons are marketing, and the "best framework" lists are affiliate SEO.

This almanac is the **public record of independent verification**. It exists because AI engineers and platform teams need a single source of truth that answers:

- Does this framework actually complete multi-step tasks end-to-end?
- What's the real ops burden of running multi-agent systems in production?
- How does it fail when tool calls are ambiguous, APIs are down, or agents race?
- What's the total cost of ownership at scale (LLM calls, retries, state storage)?
- Can I trust the vendor's GAIA or WebArena score?

## Core principles

### 1. Frozen methodology before results

The harness, judge model, prompts, scoring rubric, and adapter contracts are **fixed and published before any framework is tested**. This prevents "cherry-picking" the methodology that favors a particular framework. If a framework doesn't fit the harness, we adapt the adapter — not the rules.

### 2. Ops-first evaluation

Most benchmarks measure task completion accuracy on GAIA. We measure **what an AI engineer actually lives with**:
- Time from `pip install` to a working agent with tool registration
- Dependency conflicts when installing alongside LangChain, LlamaIndex, or Pydantic AI
- Time to debug when an agent silently loops or fails to call a tool
- Upgrade pain when version N → N+1 breaks the agent state format or tool contract
- Cost predictability when 10 agents run concurrently with retries

### 3. Raw data is always published

Every benchmark run produces a JSON file with every task, every agent trace, every tool call, every token count, every latency measurement. These raw files are published alongside the summary. If you disagree with a ranking, you can re-analyze the data yourself.

### 4. No framework is above criticism

Every framework on the roster has been through a smoke gate. Every framework has bugs — from tool call hallucination to state corruption in multi-agent orchestration. We document them honestly. A vendor relationship or sponsorship does not influence rankings. The only way a framework improves its score is by actually improving.

### 5. Living document, not a static snapshot

The almanac is updated monthly. Frameworks enter and exit the roster. Scores change as frameworks improve or degrade. The "founding edition" is a snapshot; the current edition is the truth.

## Design philosophy

### The two-bar test

Every framework must clear two bars to justify its existence:
1. **Beat the naive baseline** on task completion accuracy (single-turn LLM with basic prompting)
2. **Beat the full-capability baseline** on cost/ops burden/complexity (hand-coded agent with direct API calls)

If a framework can't do both, it has no reason to exist as infrastructure. A framework that is 5% more accurate on GAIA but 10x more complex to deploy and debug than a direct OpenAI SDK implementation is not worth adopting.

### The seven dimensions

We score every framework on seven dimensions because no single number captures "good agent infrastructure":

| Dimension | Why it matters | Agent-specific meaning |
|-----------|---------------|------------------------|
| **Accuracy** | Does it produce correct task completions? | GAIA level pass rate, WebArena task success, SWE-bench resolve rate |
| **Latency** | Does it respond fast enough for real use? | Time to complete multi-step tasks, tool call round-trip latency, planning overhead |
| **Token economics** | Does it cost what you expect? | Total LLM tokens consumed per task (including planning, reflection, retries) |
| **Scale behavior** | What happens when you 10x the load? | Concurrent agent execution, multi-agent coordination overhead, state store saturation |
| **Ops burden** | How much of your life does it consume? | Setup complexity, tool registration ceremony, state migration, debugging agent traces |
| **Developer experience** | Is it pleasant or painful to use? | Tool definition ergonomics, agent trace observability, error messages when plans fail |
| **Data sovereignty** | Can you run it yourself? Audit it? | Self-hosted agent state, local tool execution, auditability of agent decisions |

### The adapter pattern

Every framework is tested through a **CategoryAdapter** — a frozen interface that the framework must satisfy. The adapter handles setup, tool registration, task execution, and teardown. This means:
- Frameworks are tested identically
- The adapter is the only thing that changes per framework
- New frameworks can be added without changing the harness
- The adapter is published and open for review

For agent frameworks, the adapter contract specifically covers:
- **Tool registration**: How the framework exposes tools to the agent (function calling, MCP, etc.)
- **Task ingestion**: How a multi-step task is submitted to the agent
- **State management**: How the framework tracks agent state across turns
- **Execution loop**: How the framework runs the agent's plan-and-act loop
- **Result extraction**: How the final answer is retrieved from the agent

### The canary

Every benchmark batch starts with a **no-tool baseline** (the "canary"). If the benchmark leaked answers anywhere, the canary would score above zero. The canary must score exactly zero — this is a hard invariant. If it doesn't, the entire batch is invalid.

## Who this is for

- **AI engineers** evaluating which framework to build on
- **Platform engineers** deciding whether to adopt LangGraph, CrewAI, Mastra, or roll their own
- **CTOs/CIOs** making build-vs-buy decisions for autonomous agent systems with actual data
- **Open-source maintainers** who want independent benchmarking of their agent framework
- **Researchers** studying the agent orchestration and multi-agent systems landscape
- **Vendors** who want to improve their frameworks based on real evidence

## What this is NOT

- Not a marketing site for any framework vendor (OpenAI, Anthropic, Google, Microsoft, etc.)
- Not a "best of" list based on GitHub stars or VC funding rounds
- Not a tutorial on how to build agents with any framework
- Not a replacement for your own due diligence on agent safety
- Not a static document that never changes

## The "Quest"

The "Agent Engineer's Quest for the Best" is the ongoing effort to test, measure, and rank every framework on the roster. It's not a one-time effort. It's a continuous process of:
1. **Discovery** — finding new frameworks via research, community, and submissions
2. **Triage** — deciding if a framework is serious enough to enter the roster
3. **Smoke gate** — running every framework through an identical 3-turn scenario to catch bugs (register tool, execute task, verify result)
4. **Benchmark** — running standard (GAIA, WebArena, SWE-bench) + custom + stress tests
5. **Publication** — publishing raw data + summary + per-framework deep-dives
6. **Iteration** — re-testing as frameworks update, as methodology improves, as new benchmarks emerge

## How to challenge a result

If you believe a ranking or score is wrong:
1. Check the **raw results JSON** — the data is public
2. Check the **adapter implementation** — the adapter code is public
3. Check the **judge prompts** — the prompts are frozen and public
4. File an issue with a specific claim and evidence
5. We'll re-run the test or update the methodology if warranted

## Governance

- **ArdurAI** maintains the almanac and runs the Quest
- **Methodology changes** require a public RFC and at least one edition cycle of notice
- **Framework additions/removals** are decided by the triage criteria (stars, last push, community activity, seriousness)
- **Benchmark results** are machine-generated; summaries are human-reviewed for fairness
- **Conflicts of interest** are disclosed (e.g., ArdurAI contributes to some frameworks on the roster); mitigation is identical harness for all

## License

Content: CC BY 4.0  
Harness code: MIT  
Raw data: CC BY 4.0
