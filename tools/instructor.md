# Instructor

| Attribute | Value |
|-----------|-------|
| Category | Agent Frameworks |
| Type | Structured Output |
| License | Open |
| Tier | C |
| Region | Global |
| First Triaged | 2026-07-02 |
| Last Updated | 2026-07-02 |

> 11K+ stars; patching LLM clients for guaranteed Pydantic output

---

## Overview

Instructor is a structured output in the agent frameworks category.

**Language/Runtime:** Python

---

## Deep Analysis

### 1. How Is This Tool Useful?

Instructor is a lightweight Python library that patches LLM client libraries to guarantee structured output validated by Pydantic models, enabling reliable extraction of typed data from LLM responses. It works with OpenAI, Anthropic, Cohere, and other providers, adding retry logic and validation to handle malformed outputs. With 11K+ GitHub stars, it is the go-to library for structured LLM output in Python.


### 2. Gotchas of Using This Tool

Instructor's patching approach (monkey-patching client libraries) can break with provider SDK updates — when OpenAI or Anthropic update their SDKs, Instructor may need updates to match. The retry mechanism for validation failures can consume extra tokens and add latency. Complex nested schemas can be challenging for LLMs to fill correctly, even with retries.


### 3. Limitations

Instructor is Python-only (a separate TypeScript version exists but with different maintenance) and focused on structured output — it is not an agent framework and does not provide orchestration, memory, or tool calling abstractions. For those features, it is typically used alongside another framework (like Pydantic AI, which builds on similar concepts).


### 4. How Secure Is This Tool?

Instructor is MIT-licensed and runs locally. It adds no external network calls beyond your configured LLM provider. The library itself has no telemetry. Structured output validation happens locally via Pydantic. Security depends on the LLM provider you use and how you handle the extracted data.


### 5. Usefulness to General Public and Non-Technical Users

Instructor is developer-focused and requires Python proficiency plus familiarity with Pydantic models. The API is extremely simple — patch a client, define a Pydantic model, call the LLM. There is no visual builder. Non-technical users would not interact with Instructor directly but benefit from the reliable structured data it produces in applications.


### 6. What Does This Tool Solve That Others Don't?

Instructor's key differentiator is its singular focus on reliable structured output with minimal overhead — it patches existing LLM clients rather than wrapping them in a new abstraction layer. This makes it the lightest-weight option for structured extraction, composable with any other framework or application architecture. Pydantic AI builds on and extends this concept.


### 7. How Does This Tool Rank Compared to Others?

Instructor is the leading library for structured LLM output in Python (11K+ stars), widely used as a dependency in other frameworks and applications. It complements rather than competes with agent frameworks. Pydantic AI (from the same ecosystem) extends the concept to a full agent framework. Instructor's simplicity and reliability make it a staple.


### 8. How Can This Tool Be Improved? How Active Is Development?

Development is active under Jason Liu (the creator) with contributions from the community. Improvements needed include faster adaptation to provider SDK changes, more provider support, better documentation for complex schemas, and improved retry strategies. The TypeScript version (instructor-js) has less active development.


### 9. Official Maintainer Contacts

Instructor is maintained by Jason Liu and contributors (github.com/jxnl/instructor). Jason Liu is active on GitHub and Twitter/X. Contact via GitHub issues or the Instructor Discord. The library is widely used and well-maintained, with quick responses to provider API changes.


### 10. General Usage Guidance

Best for any Python project that needs reliable structured output from LLMs. Simply patch your LLM client and define Pydantic models for desired output. Use for data extraction, classification, and any task requiring typed LLM responses. Combine with other frameworks (LangChain, CrewAI) for agent capabilities. Adjust max_retries based on schema complexity.


---

## License

Content for this page is licensed CC BY 4.0 — share and adapt with attribution. Authored by Team Ardur.
