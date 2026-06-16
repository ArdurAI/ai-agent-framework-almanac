# Troubleshooting & Debugging

How to understand the codebase, debug issues, and resolve common problems when working with the agent frameworks almanac.

## Table of Contents

1. [Understanding the Codebase](#understanding-the-codebase)
2. [Common Issues](#common-issues)
3. [Debugging the Data Pipeline](#debugging-the-data-pipeline)
4. [Debugging Benchmark Runs](#debugging-benchmark-runs)
5. [FAQ](#faq)
6. [Getting Help](#getting-help)

---

## Understanding the Codebase

### High-level flow

```
Research Agents → Research Output (Markdown) →
  Python Script → roster.json (Structured Data) →
    Manual Review → Edition Markdown →
      Git Commit → GitHub Publication
```

### Key files and their roles

| File | Role | When to read it |
|------|------|-----------------|
| `README.md` | Project overview, quick reference | First thing you read |
| `INTENT.md` | Philosophy, why we do things this way | When you disagree with a decision |
| `IMPLEMENTATION.md` | How things are built, how to add frameworks | When you want to contribute |
| `TESTING.md` | Benchmark methodology, scoring, adapter contract | When you want to reproduce or challenge a result |
| `TROUBLESHOOTING.md` | This file | When something is broken |
| `architecture.md` | Stack architecture diagram | When you want to understand the big picture |
| `editions/YYYY-MM.md` | Monthly snapshot of the landscape | When you want historical data |
| `data/roster.json` | Machine-readable catalog | When you want to query or analyze the data |
| `methodology/benchmark-harness.md` | Harness specification | When you want to build an adapter or run benchmarks |
| `methodology/adapter-contract.md` | AgentAdapter interface | When you want to implement a new framework adapter |

### The data model

The almanac is fundamentally a **directed graph** of data:

```
Research findings → Framework metadata → Roster JSON → Edition Markdown → README
                                      ↓
                               Benchmark results → Per-framework pages
```

- **Research findings** are the raw output of the research swarm. They're saved in `research/` (not in the public repo).
- **Framework metadata** is extracted from research and stored in `data/roster.json`.
- **Roster JSON** is the single source of truth. Everything else derives from it.
- **Edition markdown** is human-written based on the roster and research.
- **README** is auto-generated from the roster and the latest edition.

### Understanding `data/roster.json`

This is the most important file in the repo. It is the single source of truth.

**Structure**:
```json
{
  "meta": { ... },
  "categories": {
    "agent-frameworks": {
      "name": "Agent Frameworks & Orchestration",
      "description": "...",
      "estimated_total": N,
      "tools": [
        { "name": "...", "type": "...", "license": "...", "tier": "A|B|C", "notes": "..." }
      ]
    }
  }
}
```

**How to query it**:
```bash
# Find all Tier A frameworks
jq '.categories."agent-frameworks".tools[] | select(.tier == "A") | .name' data/roster.json

# Count frameworks by tier
jq '.categories."agent-frameworks".tools | group_by(.tier) | map({tier: .[0].tier, count: length})' data/roster.json

# Find all MIT-licensed frameworks
jq '.. | objects | select(.license == "MIT") | .name' data/roster.json

# Find all Python frameworks
jq '.categories."agent-frameworks".tools[] | select(.notes | contains("Python")) | .name' data/roster.json
```

### The edition markdown

Editions are **human-written** summaries, not machine-generated. They are based on the roster but include analysis, interpretation, and narrative that a machine can't produce.

**How editions are structured**:
1. Front matter: date, research method, context
2. Landscape at a glance: summary table by tier
3. Agent frameworks section: findings, roster, analysis, multi-agent observations
4. Quest diary: what was tested this month (GAIA, WebArena, SWE-bench, smoke gates)
5. Cross-category findings: patterns that span frameworks and infrastructure tools

### The benchmark harness (separate repo)

The actual benchmark code lives in a separate repository. The almanac repo contains:
- The methodology specification
- The results (JSON + markdown)
- The adapter interface definitions

The harness repo contains:
- The runner code
- The adapter implementations (GAIAAdapter, WebArenaAdapter, SWEBenchAdapter)
- The judge model integration
- The telemetry collector
- The agent trace analyzer

**Why separate?** Because the harness is code that runs, and the almanac is data that is published. They have different lifecycles and different audiences.

## Common Issues

### Issue: `roster.json` is invalid JSON

**Symptoms**:
- `jq` fails to parse it
- GitHub Actions fails on JSON validation
- Python `json.load()` raises `JSONDecodeError`

**Diagnosis**:
```bash
python3 -c "import json; json.load(open('data/roster.json'))"
```

**Resolution**:
1. Find the line with the error: `python3 -m json.tool data/roster.json`
2. Common causes: trailing commas, unescaped quotes, incorrect nesting
3. Fix the JSON and re-validate
4. Consider using a JSON linter in your editor

### Issue: Edition markdown has broken links

**Symptoms**:
- Links to frameworks return 404
- Links to benchmarks don't exist yet
- Relative links work locally but break on GitHub

**Diagnosis**:
```bash
# Check all links in the repo
find . -name "*.md" -exec grep -oP '\[.*?\]\(.*?\)' {} + | grep -v "http" | grep -v "mailto"
```

**Resolution**:
1. For internal links, use relative paths: `../data/roster.json`
2. For external links, verify the URL is correct
3. For frameworks without a per-framework page yet, link to their GitHub repo or homepage
4. Run a link checker as part of CI

### Issue: Tier assignment is wrong

**Symptoms**:
- A framework is Tier A but has no production agent deployments
- A framework is Tier C but is widely adopted (e.g., many GitHub stars, active Discord)
- A framework's tier changed without explanation

**Diagnosis**:
1. Check the tier assignment rules in `IMPLEMENTATION.md`
2. Verify the framework's adoption, activity, and community health
3. Check the edition notes for the rationale

**Resolution**:
1. File an issue with evidence (GitHub stars, last push, production references, agent deployments)
2. The tier will be reviewed in the next edition cycle
3. Tiers are not changed mid-edition; they are updated at edition boundaries

### Issue: Benchmark results can't be reproduced

**Symptoms**:
- Running the harness produces different GAIA pass rates
- The adapter fails with a different framework version
- The judge model is unavailable
- WebArena tasks behave differently (website state changes)

**Diagnosis**:
1. Check the `results.json` metadata for the exact commit, seed, hardware, and LLM model version
2. Check if the framework version has changed since the published run
3. Verify the judge model is accessible and pinned to the same version
4. For WebArena: check if the target websites have changed (some tasks are time-sensitive)

**Resolution**:
1. Use the exact commit and dependencies from the results metadata
2. If the framework version changed, the results are for a different version — this is expected
3. If the judge model changed, that's a methodology issue — file an issue
4. For WebArena: some tasks may have shifted; note the task date in the results

### Issue: Agent framework adapter fails on tool registration

**Symptoms**:
- `register_tools()` crashes with schema errors
- Tool calls fail at runtime with "tool not found"
- Complex nested schemas are rejected by the framework

**Diagnosis**:
1. Check the framework's tool registration documentation
2. Verify the tool schema matches the framework's expected format (OpenAI functions, JSON Schema, Pydantic models, etc.)
3. Check if the framework requires specific type annotations or descriptions

**Resolution**:
1. Adapt the tool schema to the framework's native format in the adapter
2. Document the schema transformation in the adapter code
3. If the framework has a known limitation (e.g., no nested objects), note it in the framework's deep-dive page

### Issue: Benchmark results inconsistent due to non-deterministic planning

**Symptoms**:
- Same framework, same GAIA task, different results across runs
- Scores vary by more than the confidence interval
- Agent takes different tool call paths on identical tasks

**Diagnosis**:
1. Check if the framework has non-deterministic behavior (temperature > 0, async timing, random planning choices)
2. Check if the LLM backend has temperature > 0 or non-deterministic sampling
3. Check if the framework uses caching or state that persists between runs

**Resolution**:
1. Set temperature to 0 for all LLM calls in the adapter
2. Clear any persistent state between runs (agent memory, cache, state stores)
3. Record hardware specs and LLM model versions in the results metadata
4. Pin framework versions and record them
5. If non-determinism is inherent to the framework, increase the number of runs and report confidence intervals

### Issue: Multi-agent race conditions in benchmark runs

**Symptoms**:
- Concurrent agent tests fail intermittently
- Shared state is corrupted when multiple agents run simultaneously
- Agent frameworks that support multi-agent execution show inconsistent scale behavior scores

**Diagnosis**:
1. Check if the framework uses a shared state store (Redis, SQLite, in-memory dict)
2. Check if the adapter isolates state between concurrent runs
3. Check if the framework has built-in locking or isolation mechanisms

**Resolution**:
1. Use isolated state stores per run (unique database names, separate Redis namespaces)
2. Document the framework's concurrency behavior in the deep-dive page
3. If the framework is not designed for concurrent execution, adjust the scale test to sequential execution and note the limitation

### Issue: Research agent missed a framework

**Symptoms**:
- A well-known framework is not in the roster
- A framework from a specific language ecosystem is missing (e.g., Rust, Go, Elixir)
- A newly launched SDK is not in the latest edition

**Diagnosis**:
1. Check if the framework meets triage criteria in `IMPLEMENTATION.md`
2. Check if it was added in a previous edition and later removed
3. Check if it falls outside the search scope (e.g., not primarily an agent framework)

**Resolution**:
1. File an issue with the framework name, URL, and evidence of adoption/activity
2. The framework will be triaged for the next edition
3. If it meets criteria, it will be added

### Issue: Monthly update cron failed

**Symptoms**:
- No new edition was published on the 15th
- The cron job is missing from the scheduler
- The research agent timed out

**Diagnosis**:
```bash
# Check cron status
cron status

# Check the cron job list
# (use the Kimi Work cron interface)
```

**Resolution**:
1. Check if the cron job is still registered
2. Check if the research agent timed out (increase timeout if needed)
3. Manually trigger the update if the cron missed a cycle
4. Check the workspace path in the cron job configuration

### Issue: GitHub push fails

**Symptoms**:
- `git push` returns 403 or 401
- The remote is not configured
- The branch is behind origin

**Diagnosis**:
```bash
git remote -v
git status
git log --oneline -5
```

**Resolution**:
1. Verify the remote URL is correct: `git remote set-url origin https://github.com/ArdurAI/...`
2. Verify GitHub CLI auth: `gh auth status`
3. If behind origin, pull first: `git pull origin main`
4. If there are conflicts, resolve them manually

## Debugging the Data Pipeline

### Research output → roster.json

**Problem**: Research agents produce markdown, but the roster JSON is incomplete or wrong.

**Debug steps**:
1. Read the research output files in `research/` (local workspace, not in the repo)
2. Check if the Python extraction script correctly parsed the framework tables
3. Check if frameworks were dropped during triage (check the triage log)
4. Verify the JSON schema is correct

**Common bugs**:
- Framework names with special characters break JSON parsing → Escape them properly
- Frameworks with no tier get dropped → Default to Tier C if unsure
- Frameworks with no notes get empty strings → Add a minimal note
- Type field mismatch → Ensure type is one of: "Agent SDK", "Orchestration", "Multi-Agent"

### roster.json → edition markdown

**Problem**: The edition doesn't reflect the roster.

**Debug steps**:
1. Compare the framework counts in the roster vs. the edition
2. Check if the edition was written before the roster was updated
3. Check if frameworks were manually edited in the edition but not in the roster

**Resolution**:
1. The edition should be derived from the roster, not the other way around
2. If the edition has manual additions, ensure they are also in the roster
3. The edition is a human-readable summary; the roster is the source of truth

### Edition markdown → README

**Problem**: The README roster-at-a-glance doesn't match the latest edition.

**Debug steps**:
1. Check which edition is referenced in the README
2. Check if the README was updated after the edition was published

**Resolution**:
1. The README should always reference the latest edition
2. Update the README when a new edition is published
3. Consider automating README updates from the roster JSON

## Debugging Benchmark Runs

### The adapter fails

**Symptoms**:
- `setup()` crashes (missing dependencies, wrong Python version)
- `register_tools()` throws an exception (schema mismatch, unsupported tool type)
- `run()` returns nothing, hangs, or produces incorrect results
- `teardown()` fails to clean up state stores

**Debug steps**:
1. Run the adapter in isolation (without the harness)
2. Check the framework's documentation for setup requirements
3. Check if environment variables are set (API keys, model endpoints)
4. Check if the framework version matches what the adapter expects
5. Check if tool schemas match the framework's native format

**Common fixes**:
- Missing Python/Node version → Use the correct runtime version (check framework docs)
- Missing API key → Set the environment variable (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`)
- Wrong framework version → Update the adapter or pin the version
- Dependency conflict → Use a virtual environment or container
- Tool schema mismatch → Transform the schema in the adapter to match the framework's expected format

### The canary fails

**Symptoms**:
- The no-tool baseline scores above zero on GAIA or WebArena
- The canary somehow produces tool call traces
- The abstention score is not 0.000

**Debug steps**:
1. Check if the benchmark workload has leaked answers (e.g., static task cache)
2. Check if the grading pipeline has a bug
3. Check if the random seed was set
4. Check if the canary adapter is truly a no-op (no hidden tool registration)

**Resolution**:
1. If the workload leaked answers, redesign the workload or clear caches
2. If the grader has a bug, fix the grader and rerun all tests
3. This is a critical failure — the entire batch is invalid

### Results are inconsistent

**Symptoms**:
- Same framework, same test, different results across runs
- Scores vary by more than the confidence interval
- Agent behavior changes between runs (different tool calls, different reasoning)

**Debug steps**:
1. Check if the framework has non-deterministic behavior (temperature > 0, async timing, random planning)
2. Check if the LLM backend has non-deterministic sampling
3. Check if the hardware was different between runs
4. Check if the framework version changed between runs
5. Check if persistent state (cache, memory, database) carries over between runs

**Resolution**:
1. Set temperature to 0 for all LLM calls in the adapter
2. Clear all persistent state between runs (databases, caches, agent memory)
3. Record hardware specs in the results metadata
4. Pin framework versions and record them
5. Pin LLM model versions and record them

## FAQ

### Q: Why is framework X not in the roster?

A: Either it doesn't meet triage criteria, it hasn't been discovered yet, or it was removed for inactivity. File an issue with evidence and we'll triage it.

### Q: Why did framework X's score change?

A: Either the framework was updated, the methodology was refined, or we found a bug in our previous test. All three are valid reasons. Check the edition notes for the rationale. Agent frameworks are particularly volatile because LLM backend changes can silently affect reasoning quality.

### Q: Can I run the benchmarks myself?

A: Yes. The harness is published separately. Clone it, install dependencies, set your API keys, and run the adapter for the framework you want to test. See `TESTING.md` for reproducibility instructions. Note: WebArena requires a local simulator setup; SWE-bench requires Docker.

### Q: How do I challenge a ranking?

A: File an issue with specific evidence. Check the raw results JSON, the adapter code, and the judge prompts. If you find a real problem, we'll re-run or update the methodology. For agent frameworks, common challenges involve: tool call interpretation, task boundary definitions, or LLM model version differences.

### Q: Can I add a framework to the roster?

A: Yes. See `CONTRIBUTING.md` for instructions. The framework must meet triage criteria and pass the smoke gate (register tool, execute task, verify result).

### Q: Why are there separate category almanacs?

A: Each category is deep enough to warrant its own dedicated repo with per-tool pages, category-specific benchmarks, and focused community. The parent almanac is the master catalog. Agent frameworks have unique concerns (tool calling, multi-agent orchestration, safety) that deserve focused treatment.

### Q: How often are benchmarks re-run?

A: Standard: every quarter for each framework. Stress: annually. Integration: quarterly. If a framework releases a major version (e.g., LangGraph 0.3 → 0.4), we may re-run early. Agent frameworks are re-run more frequently than infrastructure tools because LLM backends change rapidly.

### Q: What's the difference between Tier A, B, and C?

A: Tier A = market leader or strongest technical merit for agent tasks. Tier B = solid option, specific use cases. Tier C = niche, early-stage, or specialized. See `IMPLEMENTATION.md` for full rules.

### Q: Can vendors sponsor the almanac?

A: No. The almanac is independently funded. Sponsorship would compromise the core mission. Vendors can improve their scores by actually improving their frameworks.

### Q: Why do some frameworks score poorly on SWE-bench?

A: SWE-bench is hard. Most agent frameworks are not designed for software engineering tasks. A low SWE-bench score doesn't mean the framework is bad — it means it's not optimized for that use case. We publish per-benchmark breakdowns so you can see where a framework excels.

### Q: What do I do if a framework fails the smoke gate?

A: Document the failure in the adapter code and the edition notes. If the failure is a bug in the framework, file an issue with the framework vendor. If the failure is an adapter bug, fix the adapter. The framework can still enter the roster as Tier C with a note about the blocker.

### Q: How do I handle frameworks that require specific cloud infrastructure?

A: The adapter should mock or simulate the cloud dependency if possible. If the framework truly requires a cloud service (e.g., a specific vector DB or message queue), document it in the adapter and ops notes. The framework's data sovereignty score will reflect this dependency.

## Getting Help

### File an issue

GitHub issues are the primary support channel. Use the appropriate template:

- **Framework request**: "Add [Framework Name] to Agent Frameworks"
- **Data correction**: "[Framework Name] metadata is wrong: [what's wrong]"
- **Benchmark challenge**: "Challenge [Framework Name] ranking on [Dimension]: [evidence]"
- **Bug report**: "[Bug description] in [file/process]"
- **Feature request**: "[Feature description] for [use case]"

### Discussion

GitHub Discussions are for:
- General questions about the almanac
- Sharing experiences with frameworks on the roster (e.g., "We run CrewAI in production and here's what we learned")
- Proposing methodology changes
- Community announcements

### Email

For private or sensitive inquiries: Use the contact info in the ArdurAI org profile.

## License

Content: CC BY 4.0
