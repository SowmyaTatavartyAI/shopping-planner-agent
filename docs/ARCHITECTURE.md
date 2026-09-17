# Architecture Guide

This document captures the architectural choices the project is intended to teach.

## 1. Workflow vs agent

A deterministic workflow is appropriate when the steps are already known.

```text
input → step A → step B → step C → output
```

An agent is appropriate when the model must decide what to do next based on observations.

```text
Goal
 ↓
Model decides next action
 ↓
Tool call
 ↓
Observation
 ↓
Model decides again
 ↓
Stop when objective is satisfied
```

The important question is not "can this be an agent?" but "does model-controlled decision-making add value here?"

## 2. Single-agent tool loop

Start here unless the problem clearly requires more.

```text
User
 ↓
Agent
 ↕
Tools
```

Good for:

- product research
- narrow shopping queries
- tasks with a small number of tools
- tasks that can be solved incrementally

## 3. Planner + executor

Use explicit planning when long-horizon tasks lose direction, contain multiple constraints, or require tracking progress across subtasks.

The planner may be:

- the same model in a separate planning call
- a structured planning phase before execution
- a dedicated planning component

It does not automatically need to be a separate agent.

```text
Goal
 ↓
Plan
 ↓
Execute step
 ↓
Observe
 ↓
Update state / replan
```

## 4. Orchestrator + workers

Use this when a task decomposes into multiple subtasks that can be handled independently.

```text
Orchestrator
   ├── worker A
   ├── worker B
   └── worker C
        ↓
    aggregation
```

Workers may be:

- deterministic functions
- API calls
- retrieval jobs
- LLM calls
- full agent loops

Only make a worker an agent if its subtask itself requires open-ended reasoning.

## 5. Supervisor + specialists

Use a supervisor when there are persistent specialist roles with distinct tool sets or responsibilities.

Example:

```text
Supervisor
   ├── Shopping specialist
   ├── Orders specialist
   └── Returns specialist
```

This is different from temporary parallel workers created for one plan.

## 6. Handoffs / peer agents

A handoff architecture does not require a permanent top-level supervisor. Responsibility can transfer from one specialist to another.

Example:

```text
Shopping agent → Order agent → Returns agent
```

This is useful when ownership naturally moves between domains.

## 7. State

State is not the same thing as the conversation transcript.

Useful categories:

### Conversational context

What the model needs to understand the current exchange.

### Structured task state

What the system needs to track explicitly.

Example:

```text
ShoppingState
    goal
    hard_constraints
    soft_preferences
    plan
    completed_steps
    products_seen
    evidence
    candidate_bundle
    budget_remaining
    pending_action
```

### Durable memory

Information that should survive beyond one task or session, if the product requirements justify it.

Avoid putting all state into natural-language messages. Explicit state makes validation, recovery, debugging, and evaluation easier.

## 8. Tools

A good tool has:

- one clear responsibility
- a precise description
- a typed input schema
- a predictable output schema
- explicit errors
- timeouts
- validation
- observability

Tool descriptions matter because the model uses them to decide whether and how to call the tool.

Avoid overlapping tools unless there is a strong reason. Overlap makes tool selection harder to debug.

## 9. Skills

A tool defines **what the agent can do**.

A skill defines **how to perform a class of tasks well**.

Example tool:

```text
get_reviews(product_id)
```

Example skill guidance:

```text
When researching a product:
1. Identify decision-critical attributes.
2. Gather evidence for those attributes.
3. Reconcile conflicting evidence.
4. Separate factual claims from subjective preferences.
5. Stop when additional research is unlikely to change the decision.
```

Skills can help avoid overloading the system prompt with every possible procedural instruction.

## 10. Retrieval / RAG

Use retrieval for knowledge the model should ground in but that does not belong permanently in context.

Examples:

- buying guides
- category-specific expertise
- policy documentation
- merchant rules

Do not use RAG as a replacement for live transactional data such as current price or inventory.

## 11. Human-in-the-loop

Separate recommendation from execution.

```text
Model proposes action
 ↓
Policy checks
 ↓
User approval if required
 ↓
Tool execution
 ↓
Verification
```

Approval requirements should be enforced by code, not merely suggested in the prompt.

## 12. Observability

For each run, capture enough information to reconstruct the trajectory:

- task ID / session ID
- user objective
- extracted constraints
- model calls
- tool selected
- tool arguments
- tool result status
- latency
- token usage
- plan revisions
- final answer
- action attempts
- errors and retries

A production debugging conversation should be able to answer:

1. What did the model know at this step?
2. What action did it choose?
3. Why was that action available?
4. What did the tool return?
5. What state changed?
6. Where did the first incorrect decision occur?

## 13. Architecture selection heuristic

Use the smallest architecture that solves the problem reliably.

```text
Known fixed steps?
    yes → deterministic workflow
    no  → single tool-using agent

Agent loses direction on long tasks?
    yes → explicit planning / replanning

Independent subtasks?
    yes → parallel workers

Do subtasks require their own reasoning loops?
    yes → agent workers
    no  → code/API workers

Permanent domain specialists with distinct ownership?
    yes → supervisor or handoff architecture
```

Complexity should be introduced in response to requirements or observed failure modes, not because multi-agent architecture sounds more sophisticated.
