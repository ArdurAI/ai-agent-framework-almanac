# Testing & Benchmarking

How the almanac tests agent frameworks, what the harness does, how scoring works, and how to reproduce results.

## Table of Contents

1. [The Three Benchmark Types](#the-three-benchmark-types)
2. [The Seven Dimensions](#the-seven-dimensions)
3. [The Harness Architecture](#the-harness-architecture)
4. [The Canary](#the-canary)
5. [Standard Benchmarks](#standard-benchmarks)
6. [PlatformOps Custom Benchmarks](#platformops-custom-benchmarks)
7. [Stress Suite](#stress-suite)
8. [Cross-Category Integration Tests](#cross-category-integration-tests)
9. [Scoring](#scoring)
10. [Reproducibility](#reproducibility)
11. [Failure Mode Taxonomy](#failure-mode-taxonomy)

---

## The Three Benchmark Types

Every framework is tested across three types of benchmarks:

| Type | Purpose | Frequency |
|------|---------|-----------|
| **Standard benchmarks** | Verify vendor claims with published test suites (GAIA, WebArena, SWE-bench) | Every benchmark run |
| **PlatformOps custom benchmarks** | Test ops reality: setup, tool registration, multi-agent reliability, failure modes | Every benchmark run |
| **Cross-category integration tests** | Test how frameworks work with vector DBs, observability tools, and security guardrails | Quarterly |

## The Seven Dimensions

Every framework is scored 0-100 on each dimension. The final score is a weighted average, but the per-dimension scores are always published.

| Dimension | Weight | What it measures | How it's tested |
|-----------|--------|-----------------|-----------------|
| **Accuracy / Quality** | 25% | Does the agent complete tasks correctly? | GAIA pass rate, WebArena task success, SWE-bench resolve rate; judge model evaluates outputs |
| **Latency** | 15% | Time to complete multi-step tasks, tool call latency, planning overhead | Instrumented measurements; p50, p95, p99 for full task completion |
| **Token Economics** | 15% | Total LLM tokens per task, cost predictability, retry overhead | Token counters from API responses; $/task computed across benchmark suites |
| **Scale Behavior** | 15% | What happens at 10x, 100x concurrent agents? | Load tests: concurrent agent execution, multi-agent coordination, state store saturation |
| **Ops Burden** | 15% | Setup complexity, tool registration ceremony, state migration, debugging | Measured setup time; smoke-gate sweep; dependency matrix; state format stability |
| **Developer Experience** | 10% | Tool definition ergonomics, agent trace quality, error clarity, docs | Structured rubric; community health metrics; time-to-first-working-agent |
| **Data Sovereignty** | 5% | Self-hosted agent state, local tool execution, auditability of agent decisions | Feature matrix; EU AI Act / GDPR / SOC 2 mapping; state exportability |

### Why these weights?

The weights reflect what an AI engineer actually cares about. Accuracy is the most important — an agent framework that produces wrong task completions or fails to use tools correctly is useless regardless of how fast or cheap it is. But ops burden is nearly as important because a framework that consumes your team's life with complex tool registration and opaque state management is not worth the accuracy gain.

Weights are reviewed annually. Changes require an RFC and a public comment period.

## The Harness Architecture

```
┌─────────────────────────────────────────┐
│  AgentAdapter (frozen contract)           │
│  ├── setup()   → install, configure       │
│  ├── register_tools() → expose tools     │
│  ├── load_task() → ingest task           │
│  ├── await_ready() → async barrier      │
│  ├── run()     → execute agent, get result│
│  └── teardown() → cleanup, measure       │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Telemetry Collector                    │
│  ├── task completion latency (p50/p95/p99)│
│  ├── token count & cost per task         │
│  ├── tool call count & success rate       │
│  ├── planning iterations & loop count     │
│  ├── error rate & failure mode taxonomy  │
│  └── ops notes (setup time, deps, bugs)   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Grading Pipeline                         │
│  ├── Deterministic grader (exact match)   │
│  ├── LLM judge (frozen prompts, SHA-256) │
│  ├── Second pass (confidence < 0.7)        │
│  └── Failure mode taxonomy               │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  Results Publisher                        │
│  ├── Raw JSON (per task, per run)        │
│  ├── Summary tables (per framework)       │
│  ├── Cross-verification analysis           │
│  └── Insight extraction                  │
└─────────────────────────────────────────┘
```

### The `await_ready()` barrier

This is where async-initialization designs get their cost measured instead of hidden. Many frameworks (graph compilation, crew warmup, agent state hydration) claim "fast" startup because the actual work happens in the background. The `await_ready()` barrier forces the framework to finish all background work before the task is run, so the true latency is measured.

### The Telemetry Collector

Every adapter call is instrumented:
- **Task completion latency**: `time.monotonic()` from task submission to final result; p50, p95, p99 computed across all tasks
- **Tool call latency**: Time per individual tool call invocation
- **Planning overhead**: Time spent in reasoning/planning loops before action
- **Tokens**: Total LLM tokens consumed (system prompt, tool definitions, reasoning traces, tool results)
- **Cost**: Token counts × provider pricing; or measured cloud spend for self-hosted frameworks
- **Tool call count**: Number of tool calls per task; success/failure rate per tool
- **Loop count**: Number of agent turns/reasoning steps per task (catches infinite loops)
- **Memory**: `psutil` or container metrics for memory usage during the run
- **CPU**: CPU utilization during agent execution
- **Errors**: Every exception, timeout, tool call failure, or unexpected result is logged with full traceback and agent trace
- **Ops notes**: Human observations about setup friction, tool registration complexity, documentation quality, error message clarity

## The Canary

The first run of every batch is the **no-tool baseline** (the "canary"). If the benchmark leaked answers anywhere, the canary would score above zero on answerable tasks.

**Canary rules**:
- The canary must score exactly **0.000** on all answerable tasks (no tool calls, no reasoning, no result)
- The canary must score exactly **0.000** on tool use tasks (it cannot use tools)
- The canary must score exactly **0.000** on SWE-bench (no code understanding, no patch)
- If the canary fails, the entire batch is invalid and must be rerun
- The canary run is published alongside the real results
- The canary adapter is a no-op: setup does nothing, register_tools does nothing, run returns empty, teardown does nothing

## Standard Benchmarks

### Agent framework benchmarks

| Benchmark | What it tests | Tasks | Source |
|-----------|-------------|-------|--------|
| **GAIA** | General AI Assistant capabilities: multi-step reasoning, tool use, file reading | 450 questions across 3 levels | [GAIA](https://huggingface.co/datasets/gaia-benchmark/GAIA) |
| **WebArena** | Web-based task completion: navigation, form filling, information retrieval | 812 tasks across 5 domains | [WebArena](https://github.com/web-arena/web-arena) |
| **SWE-bench** | Agent-based software engineering: bug understanding, patch generation, test execution | Real GitHub issues | [SWE-bench](https://github.com/princeton-nlp/SWE-bench) |

### Published vs. reproduced

Every standard benchmark ranking ships a table:

| Framework | Published Claim | Our Result | Delta | Verdict |
|-----------|----------------|------------|-------|---------|
| Framework A | 45% on GAIA | 42% on GAIA | -3% | ✅ Close |
| Framework B | "fastest WebArena" | 3rd of 8 | - | ⚠️ Misleading |
| Framework C | No claim | 38% on GAIA | N/A | — |
| Framework D | 12% on SWE-bench | 8% on SWE-bench | -4% | ⚠️ Underperforms |

## PlatformOps Custom Benchmarks

### Setup experience

**Measured**:
- Time from `pip install` / `npm install` to a working agent with registered tools
- Number of dependency conflicts when installing alongside other roster frameworks (LangGraph, CrewAI, Pydantic AI, etc.)
- Time to resolve dependency conflicts
- Number of undocumented steps required to register a tool and run a task
- Time to find the answer in the docs when stuck (e.g., "how do I enable streaming?" or "how do I set tool schemas?")

**Scored on**:
- Sub-5 minutes: 90-100
- 5-30 minutes: 70-89
- 30-60 minutes: 50-69
- 60+ minutes or unresolved: 0-49

### Smoke gate

Every framework must pass an identical 3-turn scenario before entering the roster:

```
Turn 1: Register a tool (e.g., a calculator function with schema)
Turn 2: Execute a multi-step task that requires the tool (e.g., "What is the square root of 144 plus 25?")
Turn 3: Verify the result is correct and the agent used the tool appropriately (not hallucinated)
```

**Pass criteria**:
- No crashes, no silent failures, no tool call hallucination
- Results must be deterministic (same task → same tool calls → same result)
- Framework must handle the basic case without workarounds (no manual state patching)
- Tool schema registration must work without type conversion hacks

**What the smoke gate surfaced** (from agent framework testing):
- Tool registration bugs: Framework fails to register tools with complex nested schemas
- Infinite loops: Agent gets stuck in reasoning loops when tool results are ambiguous
- State corruption: Multi-agent systems overwrite each other's state in shared stores
- Tool hallucination: Agent claims to call a tool but fabricates the result
- Cloud tethers: "Self-hosted" framework still requires cloud API for core orchestration
- Latency spread: Task completion time from 2 seconds to 5 minutes for identical tasks

### Stress suite

| Test | What it does | What it reveals |
|------|-------------|---------------|
| **Contradiction storms** | Give the agent contradictory tool results | How the framework handles conflicting information in reasoning |
| **Near-duplicate tasks** | Run hundreds of similar tasks concurrently | Deduplication of tool calls, cache behavior, state bloat |
| **Temporal paradoxes** | Tasks that change state between tool calls | Temporal reasoning accuracy, state consistency |
| **Concurrent agents** | Multiple agents writing to shared state simultaneously | Race conditions, locking, state consistency in multi-agent systems |
| **Kill-the-tool** | Crash a tool mid-execution | Recovery, error handling, retry logic, agent resilience |
| **Cost-runaway** | Run the agent at maximum scale for 1 hour | Cost predictability, token budget enforcement, billing accuracy |
| **Tool overload** | Register 50+ tools and run a task | Tool selection accuracy, context window management, schema overhead |
| **Plan deviation** | Tasks that require the agent to abandon its initial plan | Re-planning quality, sunk-cost fallacy in agent reasoning |

### Upgrade path

**Tested**:
- Can you upgrade from version N to N+1 without rewriting agent definitions?
- Are there breaking changes in the tool registration API or state format?
- Is there a migration guide for agent state or graph definitions?
- Does the framework maintain backward compatibility for tool schemas?

### Debugging experience

**Tested**:
- When the agent fails, can you find out why in <5 minutes?
- Are error messages clear and actionable (e.g., "tool X failed with Y" vs. generic "execution error")?
- Is there a trace or debug mode that shows the agent's reasoning steps?
- Are there known issues documented for common failure modes (tool hallucination, infinite loops)?
- Can you trace the execution path (which tool was called when, with what arguments)?

## Cross-Category Integration Tests

These tests run quarterly and check how frameworks work with tools from other categories in a realistic stack:

| Integration | What it tests | Tools involved |
|-------------|-------------|----------------|
| **Agent + Vector DB** | Can the agent retrieve from a vector DB and complete a RAG task? | Agent framework, vector database |
| **Agent + Observability** | Are agent traces captured by observability tools? Latency overhead? | Agent framework, observability tool |
| **Agent + Security** | Do guardrails intercept harmful tool calls? <50ms overhead? | Agent framework, security guardrail |
| **Agent + Protocol** | MCP server compliance: can the agent use an MCP tool? | Agent framework, MCP server |
| **Multi-agent + Orchestration** | Can multiple agents coordinate via A2A or shared state? | Two agent frameworks, protocol layer |

## Scoring

### Per-dimension scoring

Each dimension is scored 0-100 using a rubric. The rubric is published before any scoring happens.

**Example: Accuracy rubric**

| Score | Criteria |
|-------|----------|
| 90-100 | ≥45% on GAIA, ≥35% on WebArena, ≥10% on SWE-bench; no critical failures in stress suite |
| 80-89 | 35-45% on GAIA, 25-35% on WebArena, 5-10% on SWE-bench; minor failures in stress suite |
| 70-79 | 25-35% on GAIA, 15-25% on WebArena, 2-5% on SWE-bench; some stress suite failures |
| 60-69 | 15-25% on GAIA, 10-15% on WebArena, 1-2% on SWE-bench; frequent stress suite failures |
| 50-59 | 10-15% on GAIA, 5-10% on WebArena, <1% on SWE-bench; significant reliability issues |
| 0-49 | <10% on GAIA or fundamentally unreliable (infinite loops, tool hallucination) |

**Example: Latency rubric**

| Score | Criteria |
|-------|----------|
| 90-100 | Median task completion <30s; p95 <60s; minimal planning overhead |
| 80-89 | Median <60s; p95 <120s; acceptable planning overhead |
| 70-79 | Median <120s; p95 <300s; noticeable planning overhead |
| 60-69 | Median <300s; p95 <600s; high planning overhead |
| 50-59 | Median <600s; p95 >600s; excessive planning overhead |
| 0-49 | Median >600s or tasks frequently timeout |

### Composite score

The composite score is a weighted average of the seven dimensions:

```
Composite = (Accuracy × 0.25) + (Latency × 0.15) + (TokenEconomics × 0.15) +
            (ScaleBehavior × 0.15) + (OpsBurden × 0.15) + (DevEx × 0.10) +
            (DataSovereignty × 0.05)
```

The composite is used for ranking, but the per-dimension scores are always published. A framework with a high composite but low ops burden score is a warning sign — it may be powerful but painful to operate.

### Confidence intervals

Every score is reported with a confidence interval computed from the standard error across runs. If the intervals overlap between two frameworks, the difference is not statistically significant.

## Reproducibility

### How to reproduce a benchmark

1. Clone the benchmark harness repo (published separately)
2. Check out the exact commit used for the run (recorded in the results JSON)
3. Install the exact dependencies (lockfile is published)
4. Set up the required API keys (LLM provider, WebArena simulator, etc.)
5. Run the harness with the same adapter and same seed
6. Compare your results to the published results

### What is frozen

| Element | How it's frozen | Where to find it |
|---------|---------------|------------------|
| Judge model | Pinned model name and version | `results.json` metadata |
| Judge prompts | SHA-256 hash | `methodology/benchmark-harness.md` |
| Control variables | Documented values (temperature=0, max tokens, etc.) | `results.json` metadata |
| Random seeds | Published integer | `results.json` metadata |
| Adapter code | Published in harness repo | Separate repo |
| Test tasks | Published JSON files | `benchmarks/` directory |
| Tool definitions | Published schema and function definitions | `methodology/benchmark-harness.md` |

### What is NOT frozen (and why)

| Element | Why it changes | How we handle it |
|---------|---------------|------------------|
| Framework versions | Frameworks update | We re-run benchmarks for new versions; old results are archived |
| Provider pricing | Cloud pricing changes | Cost is computed at runtime using current pricing; historical results are annotated |
| LLM model versions | Models improve | We pin model versions per run; model changes are noted in results |
| Hardware | We may upgrade machines | Hardware spec is recorded in `results.json`; results are hardware-specific |

## Failure Mode Taxonomy

Every failure is classified into a taxonomy. This helps identify patterns across frameworks and agent categories.

| Category | Failure Modes |
|----------|--------------|
| **Setup** | `install_failed`, `dependency_conflict`, `config_error`, `missing_env_var`, `docs_incomplete`, `tool_registration_failed` |
| **Tool Execution** | `tool_call_failure`, `tool_hallucination`, `tool_schema_mismatch`, `tool_timeout`, `tool_not_found`, `tool_result_misinterpretation` |
| **Agent Reasoning** | `infinite_loop`, `plan_failure`, `reasoning_error`, `goal_misunderstanding`, `premature_termination`, `sunk_cost_fallacy` |
| **State** | `state_corruption`, `state_loss`, `state_race_condition`, `state_migration_break`, `shared_state_contamination` |
| **Query / Task** | `task_timeout`, `task_crash`, `wrong_result`, `hallucination`, `missing_tool_use`, `incorrect_tool_sequence` |
| **Scale** | `throughput_degradation`, `memory_leak`, `cpu_spike`, `agent_collision`, `state_store_saturation`, `rate_limit_hit` |
| **Ops** | `upgrade_breaking`, `undocumented_behavior`, `debug_opacity`, `community_unresponsive`, `trace_unavailable` |
| **Security** | `prompt_injection`, `tool_spoofing`, `unsafe_tool_call`, `jailbreak`, `pii_exposure`, `agent_delegation_abuse` |
| **Integration** | `mcp_noncompliant`, `a2a_noncompliant`, `auth_failure`, `protocol_mismatch`, `vector_db_sync_failure` |

## License

Content: CC BY 4.0  
Code: MIT
