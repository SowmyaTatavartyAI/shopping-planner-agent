# Shopping Planner Agent — One Pager

## Problem

A commerce platform already has rich shopping data and APIs:

- product search and catalog
- product groups and variants
- merchant offers
- price and price history
- ratings and reviews
- inventory
- buying guides / product knowledge
- cart APIs

Users still have to manually search, compare, reconcile constraints, and assemble a purchase themselves.

## Goal

Build an agentic shopping planner that turns a natural-language shopping mission into an evidence-backed, constraint-satisfying shopping plan and, with user approval, can take actions such as adding selected products to cart.

## Example task

> Build me a home-office setup for under $800. I already have a laptop. I need a monitor, keyboard, mouse, webcam, and headphones. I work from home all day, so prioritize ergonomics and good video calls.

This is deliberately different from a simple query such as:

> Find me a laptop below $500.

The second may need only a simple tool-using loop. The first requires planning, maintaining state, resolving tradeoffs, and possibly parallel research.

## Experience flow

```text
USER GOAL
    ↓
Understand intent + constraints
    ↓
Plan / identify missing information
    ↓
Search catalog
    ↓
Research promising candidates
 ┌───────────┬────────────┬───────────────┐
 price       reviews      offers/variants
 history
 └───────────┴────────────┴───────────────┘
    ↓
Update task state / replan if necessary
    ↓
Construct bundle
    ↓
Verify hard constraints deterministically
    ↓
Explain recommendation + evidence
    ↓
USER APPROVAL
    ↓
Add to cart
```

## Core design principle

**Use the model for ambiguous judgment. Use deterministic code for anything that can be checked reliably.**

Examples:

- The model decides which products deserve deeper research or whether a tradeoff is reasonable.
- Code verifies total budget, inventory, duplicate SKUs, valid IDs, and action permissions.

## Success criteria

| Layer | What to measure |
|---|---|
| Task success | Did the agent produce a valid bundle satisfying hard constraints? |
| Tool correctness | Correct tool, valid arguments, appropriate sequence, redundant calls |
| Planning / trajectory | Did it gather enough evidence without wandering? |
| Answer quality | Relevance, recommendation quality, faithfulness, completeness |
| System quality | Latency, token/tool cost, failure rate, looping, recovery success |

## Evaluation strategy

- Deterministic assertions for hard properties.
- Human-written rubrics and seed labels for subjective properties.
- LLM-as-judge only after calibration against human labels.
- Regression suites created from real failures.

The intended loop is:

```text
human failure analysis
    ↓
rubric
    ↓
eval set
    ↓
automated evals
    ↓
trace + debug
    ↓
fix
    ↓
regression suite
```

## Project versions

- **V1:** Single tool-using shopping researcher.
- **V2:** Multi-step shopping planner with structured state, retrieval, planning, and skills.
- **V3:** Action-taking commerce agent with approvals, idempotency, and safe execution.
- **V4:** Complex shopping missions with orchestration and parallel workers.

See `VERSION_PLAN.md` for details.
