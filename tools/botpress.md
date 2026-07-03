# Botpress


[![LLMOps](https://img.shields.io/badge/Also_in-LLMOps-blue)](https://github.com/ArdurAI/ai-llmops-almanac)

| Attribute | Value |
|-----------|-------|
| Category | Agent Frameworks |
| Type | Conversational |
| License | Open |
| Tier | B |
| Region | Global |
| First Triaged | 2026-07-02 |
| Last Updated | 2026-07-02 |

> Open-source dialog + LLM; modular; community extensible

---

## Overview

Botpress is a conversational in the agent frameworks category.

**Language/Runtime:** TS

---

## Deep Analysis

### 1. How Is This Tool Useful?

Botpress is an open-source conversational AI platform that combines traditional dialog management with LLM capabilities, providing a modular architecture for building, deploying, and managing chatbots. It offers a visual flow builder, NLU engine, multi-channel deployment, and a marketplace of modules and integrations. The platform has evolved from a pure chatbot builder to an LLM-augmented conversational platform.


### 2. Gotchas of Using This Tool

Botpress has undergone significant architectural changes between versions (v11 → v12 → Botpress Cloud), causing migration challenges for existing users. The TypeScript/Node.js foundation means teams need JavaScript expertise. The combination of traditional dialog management and LLM features can create confusion about which approach to use for specific use cases.


### 3. Limitations

Botpress is TypeScript/Node.js-based; there is no Python SDK. Botpress Cloud (the latest version) is a hosted platform, reducing self-hosting flexibility compared to the open-source v12. Some advanced features require the paid tiers. The ecosystem of modules and integrations, while growing, is smaller than enterprise competitors like Kore.ai.


### 4. How Secure Is This Tool?

Botpress is AGPL-licensed (open source v12) — the AGPL license requires sharing modifications, which may concern some commercial users. Botpress Cloud follows standard SaaS data handling. The self-hosted version gives full data control. Conversation data and user information flows through your configured infrastructure and channels.


### 5. Usefulness to General Public and Non-Technical Users

Botpress provides a visual flow builder that is accessible to semi-technical users for designing conversation flows. Non-technical conversation designers can use the visual editor. However, building custom integrations and advanced logic requires TypeScript development. The platform is more accessible than code-first frameworks but requires technical setup.


### 6. What Does This Tool Solve That Others Don't?

Botpress differentiates with its modular architecture and combination of traditional dialog management with LLM capabilities — teams can use deterministic flows for structured conversations and LLMs for flexible, open-ended interactions. The visual builder with code extensibility appeals to teams wanting both approaches. The multi-channel deployment is a strong feature.


### 7. How Does This Tool Rank Compared to Others?

Botpress competes with Rasa (open-source, Python), Kore.ai (enterprise), and commercial conversational AI platforms. It is less widely adopted than Rasa but has a loyal community, particularly in the Node.js ecosystem. The transition to Botpress Cloud represents a strategic shift toward managed hosting, which may affect adoption patterns.


### 8. How Can This Tool Be Improved? How Active Is Development?

Development is active under Botpress Inc. with the Cloud platform receiving regular updates. The open-source v12 maintenance has slowed as focus shifts to Cloud. Improvements needed include clearer open-source vs. Cloud positioning, migration tools, better documentation, Python SDK support, and a more active module marketplace.


### 9. Official Maintainer Contacts

Botpress is maintained by Botpress Inc. (github.com/botpress/botpress). Sylvain Perron and the team are reachable via GitHub issues, the Botpress community, and botpress.com for enterprise support. The Discord community is active for developer questions.


### 10. General Usage Guidance

Best for teams in the Node.js ecosystem building conversational AI with a mix of deterministic flows and LLM capabilities. Evaluate Botpress Cloud for managed hosting. Use the visual flow builder for conversation design and TypeScript for custom logic. Consider the AGPL license implications for commercial use of the open-source version.


---

## License

Content for this page is licensed CC BY 4.0 — share and adapt with attribution. Authored by Team Ardur.
