---
custom-width: 80
---
---
## 1. AI Fundamentals

- **AI (Artificial Intelligence):** Technology that performs tasks requiring human-like intelligence.
    
- **ML (Machine Learning):** Systems learn patterns from data.
    
- **DL (Deep Learning):** Machine learning based on multilayer neural networks.
    
- **LLM (Large Language Model):** A model that processes context and generates text or code as sequences of tokens.
    
- **Token:** A unit of text processed by a model; it can be a word, part of a word, or a symbol.
    
- **Context Window:** The amount of token-based information a model can process within a context window.
    

**Remember:** LLMs can hallucinate, invent APIs, and produce incorrect code. Always verify their output.

## 2. AI Assistants and Coding Agents

- **AI Assistant:** Answers questions and suggests code; the user often performs the actions manually.
    
- **AI Agent:** Uses a model and tools to perform actions toward a goal.
    
- **Coding Agent:** An agent designed to work with code, repositories, terminals, tests, and Git.
    
- **Tools:** External capabilities such as reading files, editing code, running commands, and controlling a browser.
    

### Agent Loop

```
Analyze → Plan → Act → Observe → Evaluate → Repeat
```

Example: An agent runs tests, analyzes a failure, modifies the code, and runs the tests again.

## 3. AI Development Tools

- **Claude Code:** Anthropic's coding agent for exploring repositories, editing code, and running commands.
    
- **OpenAI Codex:** OpenAI's AI-assisted coding tools.
    
- **Google Antigravity:** Google's AI-assisted development environment.
    
- **GitHub:** Repository hosting, version control, Pull Requests, and CI/CD workflows.
    
- **Playwright:** Browser automation and end-to-end testing.
    
- **Context7:** Provides access to library documentation through supported integrations.
    
- **Figma:** Interface design and design collaboration.
    
- **NotebookLM:** Analyzes supplied sources and creates summaries and study materials.
    

Tool capabilities depend on the product version and configuration.

## 4. Prompt Engineering

**Prompt Engineering** is the process of designing instructions to obtain a useful, accurate result.

A good prompt defines:

1. **Task** — what to do.
    
2. **Context** — relevant information.
    
3. **Requirements** — expected behavior.
    
4. **Constraints** — limitations and rules.
    
5. **Output format** — expected response.
    
6. **Definition of Done** — completion criteria.
    

Example: Implement a component using existing styles, avoid unnecessary dependencies, add tests, and report the results.

## 5. Meta Prompting

**Meta Prompting** means using AI to create or improve another prompt.

**Context-aware Prompt Enhancement** improves a prompt using information about a specific codebase, such as its files, components, and conventions.

**One-shot Coding** means attempting to complete a task with a single comprehensive prompt.

**Remember:** Better prompts improve the chances of success but do not eliminate the need for testing.

## 6. Context Engineering

**Context Engineering** is the process of selecting and organizing the information available to a model.

Context may include source code, project instructions, documentation, tests, previous decisions, and tool outputs.

**Prompt Engineering vs. Context Engineering:**

- Prompt Engineering: How should the task be formulated?
    
- Context Engineering: What information does the model need?
    

### Important Concepts

- **RAG (Retrieval-Augmented Generation):** Retrieves relevant information before providing it to the model.
    
- **Context Compression:** Summarizes a long conversation while preserving important information.
    
- **Context Caching:** Reuses eligible context to improve efficiency when supported.
    
- **Context Window Management:** Keeps the context relevant and avoids unnecessary information.
    

**Best practice:** Provide relevant files, concise instructions, and current documentation instead of overwhelming the model with unrelated information.

## 7. Project Instructions: CLAUDE.md and AGENTS.md

These files provide persistent project-specific instructions to supported AI coding tools.

They may describe:

- Project architecture.
    
- Coding conventions.
    
- Development commands.
    
- Testing procedures.
    
- Dependencies and restrictions.
    
- Security requirements.
    

Example: Follow existing patterns, avoid unnecessary dependencies, and run relevant tests before finishing.

**Remember:** Different tools may support different instruction files and loading rules.

## 8. Agent Skills

A **Skill** is a reusable set of instructions and resources for a specific workflow.

Example: A code review skill instructs an agent to inspect the diff, check edge cases, review security, and report findings.

**Benefits:** Consistency, reuse, and standardized workflows.

**Skill vs. Prompt:**

- Prompt: Describes a specific task.
    
- Skill: Defines a reusable process for performing a type of task.
    

## 9. Model Context Protocol (MCP)

**MCP (Model Context Protocol)** is a standard protocol for connecting AI applications to external tools and data sources.

Main concepts:

- **Host:** The AI application.
    
- **Client:** The component that connects to an MCP server.
    
- **MCP Server:** Exposes capabilities to the client.
    
- **Tools:** Operations the agent can invoke.
    
- **Resources:** Data made available by a server.
    
- **Prompts:** Reusable prompt templates, when supported.
    

Examples include integrations for Figma, GitHub, browser automation, and documentation.

**Skill vs. MCP:**

- Skill explains how to perform a workflow.
    
- MCP provides access to external capabilities.
    

**Security:** Use trusted servers, minimum permissions, and confirmation for sensitive actions.

## 10. Prompt Injection and Security

**Prompt Injection** is an attempt to manipulate an AI system through untrusted content, such as a webpage, issue, README, or log.

Important security practices:

- Follow the **Principle of Least Privilege**.
    
- Never expose secrets unnecessarily.
    
- Review changes before accepting them.
    
- Restrict access to files and external services.
    
- Treat external content as data, not automatically as instructions.
    
- Require additional approval for destructive or high-risk actions.
    

High-risk operations include database migrations, permission changes, production deployments, and infrastructure modifications.

## 11. Plan Mode

**Plan Mode** allows an agent to investigate a task and prepare an implementation plan before making changes.

A plan should identify:

- The goal.
    
- Relevant files and components.
    
- Implementation steps.
    
- Dependencies.
    
- Tests and verification criteria.
    
- Potential risks.
    

**Workflow:**

```
Explore → Plan → Review Plan → Implement → Test → Review
```

Plan Mode is especially useful for complex tasks and unfamiliar codebases.

## 12. Sub-Agents

A **Sub-agent** is a separate agent assigned a specific subtask.

Examples:

- Investigate project architecture.
    
- Analyze test coverage.
    
- Research documentation.
    
- Review code independently.
    

**Advantages:** Parallel work, focused context, and task specialization.

**Disadvantages:** Additional token usage, coordination overhead, conflicting changes, and information loss during handoffs.

Use sub-agents when tasks are sufficiently independent and their results can be verified separately.

## 13. Agent Teams

**Agent Teams** coordinate multiple agents working toward a shared goal.

A team might assign separate agents to research architecture, analyze requirements, implement independent modules, and review changes.

**Main risks:** Conflicting edits, duplicated work, inconsistent assumptions, and coordination costs.

**Remember:** More agents do not automatically mean better results.

## 14. Agent Harness and Frameworks

An **Agent Harness** is the execution environment that connects the model, context, tools, permissions, control loop, and verification mechanisms.

An **Agent Framework** provides reusable components for building agent-based systems.

**Multi-agent Orchestration** coordinates multiple agents, including task assignment, parallel execution, context sharing, and result integration.

```
Task → Orchestration → Models + Tools + Context → Verification → Final Result
```

Open-source models can offer more infrastructure control but may require additional hardware, maintenance, and configuration.

## 15. Spec-Driven Development (SDD)

**Spec-Driven Development** is an approach in which specifications guide planning, implementation, and verification.

```
Intent → Specification → Plan → Tasks → Implementation → Verification
```

SDD helps clarify requirements before coding and makes expected behavior easier to verify.

It is especially useful for complex features, but may be excessive for tiny changes.

## 16. GitHub Spec Kit

**GitHub Spec Kit** is a toolkit for organizing Spec-Driven Development with AI coding agents.

A typical workflow is:

```
Constitution → Specify → Clarify → Plan → Tasks → Analyze → Implement → Verify
```

The exact commands and supported integrations depend on the current version.

### Greenfield vs. Brownfield

- **Greenfield:** Building a new project from scratch.
    
- **Brownfield:** Working with an existing project and its architecture, conventions, dependencies, and technical debt.
    

For brownfield projects, inspect the codebase before defining the implementation plan.

## 17. Constitution, PRD, Specification, Plan, and Tasks

|Artifact|Purpose|
|---|---|
|**Constitution**|Defines persistent project principles and rules.|
|**PRD**|Describes product goals, users, requirements, and scope.|
|**Specification**|Defines what the feature should do and why.|
|**Clarifications**|Resolve ambiguities in requirements.|
|**Plan**|Explains how to implement the feature technically.|
|**Tasks**|Breaks the plan into actionable steps.|
|**Verification**|Checks whether the implementation meets requirements.|

**Key distinction:**

- Specification = What and why?
    
- Plan = How?
    
- Tasks = Which actions?
    
- Constitution = Which rules must always be followed?
    

## 18. PRD and Requirements Clarification

**PRD (Product Requirements Document)** describes the problem, target users, goals, scope, requirements, constraints, and acceptance criteria.

**Clarification** resolves ambiguous requirements before implementation.

Example: For a voting feature, clarify whether users can change their votes, select multiple options, or vote after a poll closes.

**Acceptance Criteria** define observable conditions that determine whether a feature is complete.

## 19. Implementation and Verification

After planning and task decomposition, the agent implements the changes and verifies the result.

Typical checks:

- **Unit Tests:** Test individual units of logic.
    
- **Integration Tests:** Test interactions between components.
    
- **E2E Tests:** Test complete user workflows.
    
- **Lint:** Detect style violations and selected code-quality problems.
    
- **Typecheck:** Check type correctness.
    
- **Build:** Verify that the application can be built.
    
- **Manual Testing:** Check actual behavior.
    

**Definition of Done** specifies the conditions required to consider a task complete.

Passing tests does not guarantee that the application is completely correct or secure.

## 20. Code Review

**Code Review** evaluates changes for correctness, maintainability, architecture, security, and potential regressions.

Check:

- Logic and edge cases.
    
- Error handling.
    
- Security and permissions.
    
- Unnecessary dependencies.
    
- Consistency with existing conventions.
    
- Test coverage.
    
- The final Git diff.
    

An AI agent can assist with review, but its findings must be evaluated.

**The developer remains responsible for the final code.**

## 21. MVP and Post-MVP

**MVP (Minimum Viable Product)** is the smallest version that provides core value and tests a key product hypothesis.

**Post-MVP** development improves the product based on feedback, observed behavior, and changing requirements.

Typical workflow:

```
Feedback → Prioritization → Updated Requirements
         → Plan → Tasks → Implementation → Verification
```

An MVP still needs appropriate security and quality controls.

## 22. Task-Master AI and Task Management

Task-management tools help transform requirements into actionable tasks and track dependencies and progress.

Typical workflow:

```
PRD → Generate Tasks → Review Dependencies
    → Refine Tasks → Implement → Verify
```

Generated tasks must be checked for completeness, duplication, and correct ordering.

## 23. NotebookLM

**NotebookLM** helps users analyze supplied sources and create summaries and other study materials, depending on available features.

Workflow:

```
Sources → Summary → Verify Against Sources → Study Questions / Audio / Slides
```

It is useful for learning and revision, but summaries may omit details or simplify important concepts.

## 24. CI/CD

**CI (Continuous Integration)** automatically checks code changes.

Typical checks include tests, lint, typechecking, and builds.

**CD (Continuous Delivery/Deployment)** automates release preparation and/or deployment.

Typical workflow:

```
Commit / Pull Request
        ↓
Lint → Typecheck → Tests → Build
        ↓
Review → Merge → Deployment
```

The exact pipeline depends on the project and team requirements.

## 25. Self-Healing CI/CD

**Self-Healing CI/CD** uses automation, potentially including AI agents, to analyze failures and propose or implement fixes.

```
CI Failure → Collect Logs → Analyze → Prepare Fix → Run Checks → Pull Request → Review
```

Best practices:

- Provide relevant logs without secrets.
    
- Restrict agent permissions.
    
- Limit automatic attempts.
    
- Run verification after every fix.
    
- Prefer a Pull Request over direct changes to protected branches.
    
- Require human review for high-risk modifications.
    

The goal is to reduce recovery time without sacrificing reliability.

## 26. Deployment and Google Cloud

**Deployment** means releasing an application into an environment where it can run and be accessed.

Typical workflow:

```
Inspect Project → Build → Configure Environment → Configure Secrets / Database → Deploy → Health Checks → Monitor
```

Before deployment, identify the stack, runtime commands, environment variables, database requirements, external APIs, and security constraints.

Google Cloud is one possible deployment platform. Cloud hosting does not automatically make an application production-ready.

## 27. Open-Source Models and Custom Harnesses

**Open-source models** can provide greater control over infrastructure and data processing, depending on their licenses and deployment model.

Consider:

- Hardware requirements.
    
- Model quality and speed.
    
- Licensing.
    
- Maintenance.
    
- Infrastructure costs.
    
- Privacy requirements.
    

A **Custom Harness** can implement project-specific permissions, tool selection, model routing, logging, and verification.

However, building a custom harness requires engineering effort and is not always better than an existing solution.

## 28. Important Workflows

### Small Code Change

```
Prompt → Inspect → Implement → Test → Review
```

### Complex Feature

```
Explore → Specify → Clarify → Plan → Tasks → Implement → Test → Review → Verify
```

### Working with an Unfamiliar Codebase

```
Inspect Instructions → Explore Architecture → Find Related Code / Tests → Plan → Implement
```

### Fixing a CI Failure

```
Read Logs → Reproduce Failure → Identify Cause → Fix → Run Checks → Review
```

### Using Sub-Agents

```
Define Independent Tasks → Delegate → Collect Findings → Validate Results → Integrate
```

### Deployment

```
Inspect → Build → Configure → Deploy → Health Checks → Monitor
```

## 29. Essential Interview Definitions

|Term|Short Definition|
|---|---|
|**LLM**|A language model that generates sequences of tokens based on context.|
|**Token**|A unit of text processed by a model.|
|**Context Window**|The amount of token-based information available within a context window.|
|**Prompt Engineering**|Designing effective instructions.|
|**Context Engineering**|Organizing information for the model.|
|**RAG**|Retrieving relevant information before generation.|
|**AI Agent**|A system that uses a model and tools to achieve a goal.|
|**Agent Loop**|The cycle of analyzing, acting, observing, and evaluating.|
|**MCP**|A protocol for connecting AI applications to tools and data.|
|**Skill**|Reusable instructions for a workflow.|
|**Sub-agent**|An agent assigned a specific subtask.|
|**Agent Team**|Multiple coordinated agents working toward a shared goal.|
|**Agent Harness**|The execution environment controlling the model, tools, and workflow.|
|**SDD**|Development guided by specifications.|
|**PRD**|A document describing product requirements.|
|**Constitution**|Persistent project principles and rules.|
|**Plan**|The technical implementation approach.|
|**Tasks**|Actionable implementation steps.|
|**MVP**|The smallest version that tests a key product hypothesis.|
|**CI**|Automated validation of code changes.|
|**CD**|Automated release preparation and/or deployment.|
|**Self-Healing CI/CD**|Automated failure analysis and repair with verification.|
|**Code Review**|Evaluation of code correctness, quality, and risks.|
|**Prompt Injection**|An attempt to manipulate an AI system through untrusted content.|
|**Least Privilege**|Giving an agent only the permissions it needs.|

## 30. Key Differences

|Concepts|Difference|
|---|---|
|Assistant vs. Agent|Answers and suggestions vs. actions through tools.|
|Prompt vs. Context|Task instructions vs. information available to the model.|
|Skill vs. MCP|Reusable workflow instructions vs. external capabilities.|
|Specification vs. Plan|Expected behavior vs. technical implementation approach.|
|Plan vs. Tasks|Overall approach vs. actionable steps.|
|Constitution vs. Specification|General project rules vs. feature-specific requirements.|
|Sub-agent vs. Agent Team|Delegated subtask vs. coordinated group of agents.|
|CI vs. CD|Automated validation vs. delivery and/or deployment.|
|Build vs. Verification|Producing a build vs. checking that requirements are met.|
|Greenfield vs. Brownfield|New project vs. existing project.|
|MVP vs. Post-MVP|Initial validation of value vs. subsequent product development.|

## Final Principle

**Reliable AI-assisted development requires more than a capable model. It requires clear requirements, relevant context, appropriate tools, limited permissions, and independent verification.**

```
Clear Goal
    + Relevant Context
    + Defined Scope
    + Acceptance Criteria
    + Safe Tool Access
    + Verification
    = Reliable AI-Assisted Development
```