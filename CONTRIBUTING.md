# Contributing to the Almanac

How to add frameworks, fix data, challenge rankings, and improve the methodology for agent framework evaluation.

## Table of Contents

1. [Ways to Contribute](#ways-to-contribute)
2. [Adding a New Framework](#adding-a-new-framework)
3. [Fixing Data](#fixing-data)
4. [Challenging a Ranking](#challenging-a-ranking)
5. [Improving the Methodology](#improving-the-methodology)
6. [Code of Conduct](#code-of-conduct)
7. [License](#license)

---

## Ways to Contribute

You can contribute to the almanac in several ways:

| Contribution Type | What you do | Impact |
|-------------------|-------------|--------|
| **Add a framework** | File an issue with a new agent framework | Expands the roster |
| **Fix data** | Correct incorrect metadata | Improves accuracy |
| **Challenge a ranking** | Provide evidence that a score is wrong | Drives quality |
| **Share experience** | Write about using a framework in production | Adds real-world context |
| **Improve methodology** | Propose a better benchmark or scoring rubric for agents | Improves fairness |
| **Build an adapter** | Implement the adapter for a new framework (GAIA, WebArena, SWE-bench) | Enables testing |
| **Review an edition** | Proofread, fact-check, suggest improvements | Improves quality |
| **Spread the word** | Share the almanac with your community | Grows the ecosystem |

## Adding a New Framework

### Before you submit

Check if the framework meets the triage criteria:

1. **Seriousness**: Is it a real framework with real users, not a toy or demo? Evidence: production deployments, GitHub issues from real users, conference talks, funding.
2. **Activity**: Has it had a push or release in the last 6 months?
3. **Documentation**: Does it have a README, docs, or landing page explaining how to build an agent, register tools, and run tasks?
4. **Accessibility**: Is it testable (open source, free tier, or evaluation license)?
5. **Scope**: Does it fit the agent framework category? Must have agent orchestration, tool calling, or multi-agent capabilities. A general-purpose LLM library without agent features does not qualify.

### How to submit

**Option 1: GitHub Issue (preferred)**

File an issue with this template:

```markdown
## Framework Request: [Framework Name]

### Category
Agent Frameworks & Orchestration

### Framework URL
[GitHub repo or homepage URL]

### License
[e.g., MIT, Apache-2.0, Proprietary]

### Type
[Agent SDK | Orchestration | Multi-Agent]

### Language
[Python, TypeScript, Java, Go, Rust, etc.]

### Description
[What does it do? One paragraph. Focus on agent capabilities: tool calling, state management, multi-agent support, etc.]

### Why it should be on the roster
[Evidence of adoption, production usage, or technical merit.]

### Evidence
- GitHub stars: [N]
- Last release: [date]
- Notable users: [companies, if known]
- Funding: [amount, if known]
- Tool calling mechanism: [OpenAI functions, MCP, JSON Schema, etc.]

### Tier suggestion
[A, B, or C — and why]
```

**Option 2: Pull Request**

If you want to add the framework directly:

1. Fork the repo
2. Edit `data/roster.json` to add the framework to the `agent-frameworks` category
3. Update `README.md` if the framework is Tier A
4. Submit a PR with the same template as above

### What happens after submission

1. **Triage**: We check if the framework meets criteria (within 7 days)
2. **Smoke gate**: We run the framework through the agent-specific 3-turn scenario (within 14 days)
   - Turn 1: Register a tool
   - Turn 2: Execute a multi-step task
   - Turn 3: Verify the result
3. **Decision**: Accepted, rejected, or deferred with a note
4. **Publication**: If accepted, it appears in the next edition

## Fixing Data

### If you find incorrect metadata

File an issue with:

```markdown
## Data Correction: [Framework Name]

### Current (incorrect) data
[What does the roster say?]

### Correct data
[What should it say?]

### Evidence
[Link to the source that proves the correct data.]
```

### Common corrections

| Field | Common errors | How to verify |
|-------|--------------|---------------|
| License | Wrong SPDX identifier | Check the repo's LICENSE file |
| Stars | Out of date | Check the GitHub API |
| Last push | Wrong date | Check the GitHub repo |
| Tier | Wrong tier | Check the tier rules in IMPLEMENTATION.md |
| Type | Wrong classification | Check if it's primarily an SDK, orchestration layer, or multi-agent system |
| Notes | Outdated description | Check the framework's homepage/docs |

### What happens after submission

Data corrections are reviewed and applied in the next edition cycle. We don't edit editions retroactively; we correct the data and note it in the next edition.

## Challenging a Ranking

### If you believe a score is wrong

File an issue with:

```markdown
## Challenge: [Framework Name] on [Dimension]

### Current score
[What does the almanac say?]

### Your evidence
[What data do you have?]

### What you did to verify
[Steps you took to reproduce or verify.]

### Suggested resolution
[What should change? Re-run? Different score? Methodology update?]
```

### What evidence is valid

| Evidence Type | Strength | Example |
|---------------|----------|---------|
| Raw results JSON analysis | Strong | "I re-analyzed the JSON and found that 12% of tool calls were misclassified as failures" |
| Independent reproduction | Strong | "I ran the GAIA harness and got 48% vs. your 42%" |
| Adapter bug documentation | Strong | "The adapter doesn't handle the framework's async tool registration, causing false failures" |
| Documentation of a framework bug | Medium | "The framework has a known bug that affects tool schema parsing on this test" |
| Vendor claim | Weak | "The vendor says 50% on GAIA" — but we already test vendor claims |
| Anecdote | Weak | "It worked for me" — not reproducible |

### Common challenge scenarios for agent frameworks

| Scenario | What to check | Likely outcome |
|----------|-------------|----------------|
| GAIA score seems low | Check if the adapter uses the correct tool set (calculator, web search, file reader) | Adapter may be missing tools the framework needs |
| WebArena score seems high | Check if the framework has built-in browser automation that's not generalizable | Score may be valid but not representative of general agent capability |
| Latency score seems off | Check if the adapter includes `await_ready()` time and planning overhead | Adapter may be measuring only the final LLM call |
| Token economics seem wrong | Check if the adapter counts system prompts, tool definitions, and reasoning traces | Adapter may only count output tokens |
| Scale behavior seems wrong | Check if the test used concurrent agents or sequential | Framework may not support true concurrency |

### What happens after submission

1. **Review**: We review the evidence (within 7 days)
2. **Reproduction**: If the claim is reproducible, we re-run the test
3. **Update**: If the re-run confirms the challenge, we update the score
4. **Publication**: The update appears in the next edition

## Improving the Methodology

### If you want to propose a methodology change

File an issue with:

```markdown
## Methodology Proposal: [Title]

### Current state
[What does the methodology say now?]

### Proposed change
[What should it say?]

### Rationale
[Why is this better? What problem does it solve?]

### Impact
[Which frameworks would be affected?]

### Backward compatibility
[Can old results be re-scored with the new method?]
```

### Methodology change process

1. **RFC**: The proposal is posted as an RFC for public comment (30 days)
2. **Discussion**: Community feedback is collected
3. **Decision**: ArdurAI makes the final decision based on feedback
4. **Announcement**: If accepted, a public announcement is made with a transition plan
5. **Implementation**: The change is implemented in the next edition cycle
6. **Re-run**: Affected benchmarks are re-run with the new methodology

### What kinds of changes are accepted

| Change Type | Likelihood | Example |
|-------------|------------|---------|
| Bug fix in harness | High | "The adapter incorrectly handles tool timeout for async frameworks" |
| New benchmark | Medium | "Add a new benchmark for multi-agent coordination (e.g., collaborative task completion)" |
| Weight adjustment | Medium | "Increase ops burden weight from 15% to 20% for agent frameworks" |
| New dimension | Low | "Add a 'safety' dimension for agent frameworks" |
| Remove dimension | Very low | "Remove latency as a dimension" |

### What kinds of changes are rejected

- Changes that favor a specific vendor or framework
- Changes that reduce reproducibility (e.g., removing frozen prompts)
- Changes that increase complexity without clear benefit
- Changes that are not backward-compatible without a migration plan

## Code of Conduct

### Be respectful

This is a collaborative project. Treat others with respect, even when you disagree.

### Be evidence-based

Claims should be backed by evidence. "I think X is better" is not enough. "I measured X and found Y" is. For agent frameworks, evidence includes: benchmark traces, adapter logs, token counts, and reproducible setup steps.

### Be constructive

Criticism is welcome if it's constructive. "This is wrong" is not helpful. "This is wrong because the adapter doesn't register nested tool schemas, and here's a fix" is.

### Be patient

The almanac is maintained by a small team. Responses may take time. Repeated pings are not helpful.

### No spam

Don't submit the same framework multiple times. Don't submit frameworks that clearly don't meet criteria. Don't use the almanac for marketing.

## License

By contributing to the almanac, you agree that your contributions are licensed under CC BY 4.0 for content and MIT for code.

## Attribution

Contributors are recognized in the edition notes. If you make a significant contribution (e.g., adding 5+ frameworks, fixing major data issues, improving methodology, building an adapter for a Tier A framework), you will be listed as a contributor in the next edition.

## License

Content: CC BY 4.0  
Code: MIT
