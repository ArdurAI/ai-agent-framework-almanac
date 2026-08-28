# Letta


[![Infrastructure](https://img.shields.io/badge/Also_in-Infrastructure-blue)](https://github.com/ArdurAI/ai-infrastructure-almanac)

| Attribute | Value |
|-----------|-------|
| Category | Agent Frameworks |
| Type | Stateful Agent |
| License | Open |
| Tier | A |
| Region | Global |
| First Triaged | 2026-07-02 |
| Last Updated | 2026-07-02 |

> 22.3K+ stars; memory-rich agents; state/session management

---

## Overview

Letta is a stateful agent in the agent frameworks category.

**Language/Runtime:** Python, TS

---

## Deep Analysis

### Daily monitoring update — 2026-08-28

- **Adoption signal:** GitHub stars moved from 23,829 to 24,470 (+641). Track 24,470 as the current monitoring baseline because this crossed the >500 daily-change threshold.

### 1. How Is This Tool Useful?

Letta (formerly MemGPT) is an open-source framework for building stateful, memory-rich agents that maintain long-term memory and state across sessions. It implements a memory architecture inspired by operating system design (main context, external archival, recall memory) enabling agents to remember and reason over extended conversations. With 22K+ GitHub stars, it is a leading solution for agents that need persistent memory.


### 2. Gotchas of Using This Tool

Letta's memory architecture adds complexity — developers must understand memory tiers (core memory, archival memory, recall memory) and how the agent manages them. The self-editing memory model (agents update their own memory) can produce unexpected behavior if not carefully monitored. Performance overhead from memory management operations adds latency to each interaction.


### 3. Limitations

Letta is Python and TypeScript but the core framework is Python-first; TypeScript support is secondary. The memory-centric design makes it less suited for stateless or single-turn use cases. Self-hosting the full Letta server (with vector DB, database, and agent runtime) requires infrastructure setup. The free open-source version lacks the managed deployment features of Letta Cloud.


### 4. How Secure Is This Tool?

Letta is Apache 2.0 licensed and runs locally or self-hosted. Agent memory (conversation history, archival data) is stored in your configured databases and vector stores, giving full control over sensitive data. Letta Cloud adds commercial data handling — review the terms for compliance. The framework introduces no telemetry beyond standard logging.


### 5. Usefulness to General Public and Non-Technical Users

Letta is developer-focused but its REST API and SDKs make integration accessible. The concept of agents with memory is intuitive for non-technical stakeholders to understand. The Letta Playground provides a UI for testing agents. Building and configuring memory schemas requires Python/TypeScript proficiency.


### 6. What Does This Tool Solve That Others Don't?

Letta's key differentiator is its OS-inspired memory architecture — no other major framework provides the same depth of memory management with self-editing memory blocks, archival search, and recall logging. This makes it uniquely suited for use cases requiring long-term agent memory: personal assistants, companions, and research agents that accumulate knowledge over time.


### 7. How Does This Tool Rank Compared to Others?

Letta is the leading framework for stateful, memory-rich agents, with 22K+ stars and strong academic roots (originating from UC Berkeley research). It competes with general-purpose frameworks (LangChain, CrewAI) for agent use cases but is the go-to for memory-intensive applications. The MemGPT research paper has been highly influential in the agent memory space.


### 8. How Can This Tool Be Improved? How Active Is Development?

Development is active under Letta (the company) with frequent releases adding features like multi-agent memory sharing, improved memory management, and deployment tooling. Improvements needed include better documentation for memory schema design, TypeScript feature parity, more pre-built memory templates, and clearer guidance on production deployment.


### 9. Official Maintainer Contacts

Letta is maintained by Letta (github.com/letta-ai/letta, formerly cpacker/MemGPT). Charles Packer and Sarah Wooders (UC Berkeley) are the founders. Contact via GitHub issues or the Letta Discord. Enterprise support is available via letta.com.


### 10. General Usage Guidance

Best for use cases requiring agents with long-term memory — personal assistants, research agents, customer support agents that remember user history. Start with the ADE (Agent Development Environment) for prototyping. Carefully design memory schemas for your use case. Use archival memory for large knowledge bases. Evaluate Letta Cloud for managed deployment.


---

## License

Content for this page is licensed CC BY 4.0 — share and adapt with attribution. Authored by Team Ardur.
