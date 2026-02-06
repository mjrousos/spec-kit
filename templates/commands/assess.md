---
description: Assess an existing codebase and provide detailed reports on its user experience, business logic, and technical architecture. This optional command creates context to be used while working in an existing codebase.
tools: ['edit', 'execute', 'read', 'search', 'web/fetch', 'agent']
---

# Assess Codebase

Your task is to leverage sub-agents to run several assessment workflows of the current solution codebase.

To do this, use the #runSubagent tool to run the following two sub-agent workflows. Each sub-agent will perform a specific type of analysis on the solution and generate a report.

**IMPORTANT**: For prompt files executed by sub-agents, do not simply summarize the prompts for the sub-agents. Instead, **share the file paths with instructions that the sub-agents should read the files themselves so that they can see the full details and requirements**.

## Sub-Agent Workflows

**Sub-Agent Workflow 1: Business Logic Analysis**

1. Run the `assess-business-logic.prompt.md` prompt with the codebase available as context.
2. Run the `assess-business-logic-validation.prompt.md` prompt with the generated business logic summary (`.specify/memory/business-logic-summary.md`) and the codebase as context.
3. Append this text to the end of the business logic summary file (`.specify/memory/business-logic-summary.md`): `Business logic analysis and validation completed.`

**Sub-Agent Workflow 2: Technical Architecture Analysis**

1. Run the `assess-technical-architecture.prompt.md` prompt with the codebase available as context.
2. Run the `assess-technical-architecture-validation.prompt.md` prompt with the generated architecture assessment (`.specify/memory/technical-architecture.md`) and the codebase as context.
3. Append this text to the end of `.specify/memory/technical-architecture.md`: `Technical architecture analysis and validation completed.`
