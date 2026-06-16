# Implementation Guide

How the almanac is built, how to add a framework, how to update an edition, and how the data pipeline works.

## Table of Contents

1. [Repository Structure](#repository-structure)
2. [The Data Pipeline](#the-data-pipeline)
3. [Adding a New Framework](#adding-a-new-framework)
4. [Updating an Edition](#updating-an-edition)
5. [The Roster JSON Schema](#the-roster-json-schema)
6. [Directory Conventions](#directory-conventions)
7. [Building the Adapter](#building-the-adapter)
8. [Automation](#automation)

---

## Repository Structure

```
ai-agent-framework-almanac/
├── README.md                          # Project overview + roster at a glance
├── INTENT.md                          # Philosophy, design principles, governance
├── IMPLEMENTATION.md                  # This file
├── TESTING.md                         # Benchmark methodology, harness details
├── TROUBLESHOOTING.md                 # Common issues, debugging, FAQ
├── CONTRIBUTING.md                    # How to contribute
├── architecture.md                    # Stack architecture + test philosophy
├── SETUP.md                           # How to push to GitHub
├── .gitignore
│
├── editions/                          # Monthly editions
│   └── 2026-06.md                   # Founding edition
│
├── benchmarks/                        # Benchmark results (rolling)
│   ├── gaia/
│   │   └── gaia-<framework>-<date>.json
│   ├── webarena/
│   │   └── webarena-<framework>-<date>.json
│   └── swe-bench/
│       └── swe-bench-<framework>-<date>.json
│
├── methodology/
│   └── benchmark-harness.md         # Detailed harness spec
│   └── adapter-contract.md          # AgentAdapter interface specification
│
├── data/
│   └── roster.json                  # Machine-readable catalog (82 tools)
│
├── tools/                             # Per-framework deep-dive pages
│   └── (populated as deep-dives are written)
│
└── assets/                            # Charts, diagrams, screenshots
    └── (populated by editions)
```

## The Data Pipeline

The almanac data flows through four stages:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Discovery      │────▶│  Triage         │────▶│  Research       │────▶│  Publication    │
│  (find tools)   │     │  (decide entry) │     │  (deep dive)    │     │  (write edition) │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Stage 1: Discovery

Frameworks are discovered through:
- **Monthly research swarm**: 8-10 parallel agents search for new frameworks in agent orchestration, multi-agent systems, and tool-calling platforms
- **Community submissions**: Issues, PRs, email, social media
- **Vendor announcements**: SDK releases (OpenAI Agents SDK, Google ADK, Claude Agent SDK), major version bumps (LangGraph, CrewAI), funding rounds
- **GitHub trending**: New repos with significant star growth in agent-related categories
- **Conference talks**: Papers, demos, blog posts from NeurIPS, ICML, AI engineering conferences

### Stage 2: Triage

A framework enters the roster if it meets ALL of these criteria:
1. **Seriousness**: Not a toy/demo. Must have a real use case, real users, or real funding. A single Jupyter notebook with 50 stars is not enough.
2. **Activity**: Last push or release within 6 months. Exceptions for "stable/mature" frameworks with active maintenance.
3. **Documentation**: Must have a README, docs, or at least a landing page explaining what it does, how to register tools, and how to run an agent.
4. **Accessibility**: Must be accessible to test (open source, free tier, or evaluation license available). Frameworks that require enterprise contracts with no evaluation path are excluded.
5. **Scope**: Must fit the agent framework category. A general-purpose LLM library does not enter unless it has specific agent/orchestration capabilities.

A framework is **excluded** if:
- It's a fork with no meaningful divergence from the parent (e.g., a thin wrapper around LangChain with no new orchestration logic)
- It's a wrapper around another framework with no added value (e.g., a UI layer that doesn't change the agent runtime)
- It has no users, no community, and no evidence of real-world agent deployments
- It requires an enterprise-only license with no evaluation path

### Stage 3: Research

For each new framework, we collect:
- Name, type (orchestration, multi-agent, agent SDK), license, language, GitHub URL, stars
- Last push date, release cadence
- Key features: tool calling mechanism, state management, multi-agent support, observability, safety features
- Known bugs and sharp edges (from smoke gate): tool registration failures, infinite loops, state corruption
- Community health: issues, PRs, maintainer responsiveness, Discord/Slack activity
- Ecosystem: MCP compatibility, A2A protocol support, integration with vector DBs, observability tools

This data is stored in `data/roster.json` and summarized in the edition.

### Stage 4: Publication

The edition is a markdown file that includes:
- Landscape at a glance table
- Per-tier findings and trends
- New frameworks added and frameworks removed
- Notable releases and acquisitions
- Quest diary (what was tested this month): GAIA runs, WebArena passes, SWE-bench attempts
- Multi-agent system observations: which frameworks handle agent collisions well, which ones degrade under concurrency

## Adding a New Framework

### Step 1: Verify the framework meets triage criteria

Check: seriousness, activity, documentation, accessibility, scope.

### Step 2: Add to the roster JSON

Edit `data/roster.json` and add the framework to the `agent-frameworks` category:

```json
{
  "name": "FrameworkName",
  "type": "Agent SDK | Orchestration | Multi-Agent",
  "license": "License",
  "region": "Region",
  "tier": "A|B|C",
  "notes": "One-line description and key differentiators (e.g., 'MCP-native, graph-based orchestration, TypeScript-first')"
}
```

**Tier assignment rules**:
- **Tier A**: Market leader, widest adoption, or strongest technical merit for agent tasks. Must be actively maintained and have real production agent deployments. Examples: LangGraph, CrewAI, Mastra, Pydantic AI.
- **Tier B**: Solid option, actively maintained, but not the market leader. Good for specific use cases (e.g., Java/Spring ecosystems, specific cloud platforms). Examples: Genkit, Spring AI, LangChain4j.
- **Tier C**: Niche, early-stage, or specialized. Worth knowing about but not a default choice. Examples: ROSA, RAI, LeRobot, TARS.

### Step 3: Update the edition

Add the framework to the `agent-frameworks` section in `editions/YYYY-MM.md`. If the framework is Tier A, add it to the roster-at-a-glance table in the README.

### Step 4: Run the smoke gate

Before the framework is officially "in," it must pass the smoke gate (see TESTING.md). The agent-specific smoke gate is:

```
Turn 1: Register a tool (e.g., a calculator function)
Turn 2: Execute a multi-step task that requires the tool
Turn 3: Verify the result is correct and the agent used the tool appropriately
```

If it fails, document the failure in the edition and assign it to Tier C with a note about the blocker.

## Updating an Edition

### Monthly update checklist

```
□ Check for new frameworks (discovery phase)
□ Triage new frameworks (add to roster or reject)
□ Update metadata for existing frameworks (stars, last push, releases)
□ Flag frameworks for removal (dead/abandoned)
□ Run smoke gate for new frameworks
□ Run benchmark updates for re-tested frameworks (GAIA, WebArena, SWE-bench)
□ Draft the edition markdown
□ Update README roster-at-a-glance
□ Commit and push
```

### Edition markdown template

```markdown
# Edition YYYY-MM — [Title]

*Research conducted YYYY-MM-DD. [Context about this month].*

## The landscape at a glance

| Tier | Count | New This Month | Notable Changes |
|------|-------|----------------|-----------------|
| A    | N     | +M             | [what changed]  |
| B    | N     | +M             | [what changed]  |
| C    | N     | +M             | [what changed]  |

## Agent Frameworks — [Theme]

### Tier A roster
[table]

### Findings
[bullets]

## Quest diary — [Month] [Year]

- GAIA: [frameworks tested, pass rates]
- WebArena: [frameworks tested, task success rates]
- SWE-bench: [frameworks tested, resolve rates]
- Smoke gate: [new frameworks, failures]

## Multi-agent observations

[What we learned about multi-agent execution, concurrency, and coordination this month]

## Coming next month

[what's planned]

## License
Content is licensed CC BY 4.0.
```

## The Roster JSON Schema

```json
{
  "meta": {
    "name": "Agent Frameworks & Orchestration Almanac Roster",
    "version": "YYYY-MM",
    "generated_at": "ISO-8601 timestamp",
    "total_tools": number,
    "categories": number,
    "research_method": "description"
  },
  "categories": {
    "agent-frameworks": {
      "name": "Agent Frameworks & Orchestration",
      "description": "Frameworks and platforms for building, deploying, and orchestrating AI agents with tool calling, multi-step reasoning, and multi-agent coordination",
      "estimated_total": number,
      "tools": [
        {
          "name": "Framework Name",
          "type": "Agent SDK | Orchestration | Multi-Agent",
          "license": "License",
          "region": "Region",
          "tier": "A|B|C",
          "notes": "Description"
        }
      ]
    }
  }
}
```

**Field definitions**:
- `name`: The framework's common name. Use the name the framework calls itself.
- `type`: What kind of framework is it? (e.g., "Agent SDK", "Orchestration Platform", "Multi-Agent Framework", "Agent SDK with Graph Orchestration")
- `license`: The primary license. Use SPDX identifiers where possible.
- `region`: Where the framework is primarily developed (US, EU, China, Global, etc.)
- `tier`: A, B, or C (see tier rules above)
- `notes`: One-line description with key differentiators. Keep under 100 chars. Mention tool-calling style, state model, or language if notable.

## Directory Conventions

### `editions/`
- One file per month: `YYYY-MM.md`
- Never delete old editions. The history is part of the record.
- New editions are appended; old editions are never rewritten.

### `data/`
- `roster.json` is the single source of truth for the framework catalog.
- It is machine-generated from the research process.
- It should be valid JSON at all times.

### `benchmarks/`
- Subdirectories per benchmark suite: `gaia/`, `webarena/`, `swe-bench/`
- One file per benchmark run: `<suite>-<framework>-<date>.md`
- Raw JSON files alongside the markdown: `<suite>-<framework>-<date>.json`
- Raw data is never deleted. It is the audit trail.

### `tools/`
- One file per framework: `<name>.md`
- Contains deep-dive analysis: setup experience, benchmark results, agent trace examples, bug notes, comparison with peers
- Populated as deep-dives are written (not all frameworks have a page immediately)

### `assets/`
- Images, charts, diagrams referenced by editions and benchmarks
- Named descriptively: `gaia-pass-rate-2026-06.png`, `webarena-latency-2026-06.png`, `agent-framework-landscape-2026-06.png`

### `methodology/`
- The benchmark harness specification and adapter contract
- Frozen before any results are generated
- Changes require an RFC and a public announcement

## Building the Adapter

When a new framework is added to the roster and is ready for benchmarking, an adapter must be built. The adapter is the bridge between the framework's API and the harness's fixed interface.

### The AgentAdapter contract

```python
class AgentAdapter:
    def setup(self) -> None:
        """Install, configure, and start the framework. Set up any required state stores."""
        pass
    
    def register_tools(self, tools: List[Tool]) -> None:
        """Register the available tools with the framework's agent runtime."""
        pass
    
    def load_task(self, task: Task) -> None:
        """Ingest the task into the framework."""
        pass
    
    def await_ready(self) -> None:
        """Wait for async initialization to complete. Measure lag."""
        pass
    
    def run(self, task: Task) -> AgentResult:
        """Run the agent on the task and return the result."""
        pass
    
    def teardown(self) -> None:
        """Clean up, measure resource usage, tear down state stores."""
        pass
```

### Adapter rules

1. The adapter must be **pure** — it should not modify the framework's behavior, only interface with it.
2. The adapter must be **documented** — every step should be explainable in plain English.
3. The adapter must be **reproducible** — running it twice on the same machine should produce the same setup.
4. The adapter must be **isolated** — it should not depend on other frameworks' adapters.
5. The adapter code is **published** in the benchmark harness repo (separate from the almanac repo).

### Example adapter (pseudocode)

```python
class GAIAAdapter(AgentAdapter):
    def __init__(self, framework_name, config):
        self.framework = framework_name
        self.config = config
    
    def setup(self):
        if self.framework == "langgraph":
            self.graph = build_graph(self.config)
        elif self.framework == "crewai":
            self.crew = Crew(agents=self.config.agents, tasks=self.config.tasks)
        elif self.framework == "openai-agents":
            self.agent = Agent(tools=self.config.tools, model=self.config.model)
    
    def register_tools(self, tools):
        for tool in tools:
            self.framework.register_tool(tool.name, tool.function, tool.description)
    
    def load_task(self, task):
        self.current_task = task
    
    def await_ready(self):
        # Wait for any async initialization (e.g., graph compilation, crew warmup)
        pass
    
    def run(self, task):
        if self.framework == "langgraph":
            return self.graph.invoke({"task": task.question})
        elif self.framework == "crewai":
            return self.crew.kickoff(inputs={"task": task.question})
        elif self.framework == "openai-agents":
            return self.agent.run(task.question)
    
    def teardown(self):
        # Clean up any persistent state stores or temporary files
        pass
```

### Adapter variants

#### GAIA Adapter
- Focuses on general reasoning and tool use (calculator, web search, file reading)
- Must handle tasks that require multiple tool calls in sequence
- Measures: task success rate, number of tool calls per task, token consumption

#### WebArena Adapter
- Focuses on web-based task completion (navigation, form filling, information retrieval)
- Must handle tasks that require browser interaction or simulated web actions
- Measures: task success rate, navigation efficiency, failure modes (stuck on page, wrong action)

#### SWE-bench Agent Adapter
- Focuses on software engineering tasks (code understanding, bug fixing, test execution)
- Must handle tasks that require reading code, writing patches, and running tests
- Measures: resolve rate (patch passes tests), number of iterations, tool use accuracy

## Automation

### Monthly update cron

The monthly update is run by a scheduled job:
- **Trigger**: `cron` expression `0 7 15 * *` (monthly, 15th at 7:00 AM)
- **Action**: Runs a research agent to discover new frameworks, update metadata, and draft the next edition
- **Output**: Commits to the repo with the updated roster and new edition

### GitHub Actions (optional)

For automatic metadata refresh (GitHub stars, last push dates), a GitHub Actions workflow can be configured:

```yaml
name: Monthly Metadata Refresh
on:
  schedule:
    - cron: '0 7 1 * *'
  workflow_dispatch:
jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Refresh metadata
        run: python scripts/refresh_metadata.py
      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add -A
          git commit -m "Monthly metadata refresh: $(date +%Y-%m)" || echo "No changes"
          git push
```

## License

Content: CC BY 4.0  
Code: MIT
