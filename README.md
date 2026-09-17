# Shopping Planner Agent

A production-oriented agentic commerce project built to explore how a shopping assistant can turn a natural-language shopping mission into an evidence-backed, constraint-satisfying plan and, with user approval, take actions such as adding selected products to cart.

## Project goals

This repo is intentionally versioned so each stage introduces a new agent-system concept while keeping one coherent application.

- **V1:** Single tool-using shopping researcher
- **V2:** Multi-step shopping planner with structured state, retrieval, and skills
- **V3:** Action-taking commerce agent with approvals and safe execution
- **V4:** Complex shopping missions with orchestration and parallel workers

## Example user goal

> Build me a home-office setup for under $800. I already have a laptop. I need a monitor, keyboard, mouse, webcam, and headphones. I work from home all day, so prioritize ergonomics and good video calls.

## Design principle

**Use the model for ambiguous judgment. Use deterministic code for anything that can be checked reliably.**

Examples:

- The model decides which products deserve deeper research or whether a tradeoff is reasonable.
- Code verifies hard constraints such as total budget, stock, duplicate SKUs, valid IDs, and action permissions.

## Repository structure

```text
shopping-planner-agent/
├── README.md
├── docs/
│   ├── ONE_PAGER.md
│   ├── ARCHITECTURE.md
│   ├── VERSION_PLAN.md
│   ├── EVALUATION.md
│   └── FAILURE_MODES.md
├── src/
├── evals/
├── data/
└── tests/
```

## What this project is meant to teach

- Agent loop design
- Tool schemas and tool selection
- State vs conversation context
- Planning and replanning
- Retrieval / RAG
- Skills and procedural context
- Tracing and observability
- Agent evaluation
- Human-in-the-loop actions
- Idempotency and failure recovery
- Orchestrator/worker patterns
- Latency, cost, and reliability tradeoffs

See `docs/ONE_PAGER.md` and `docs/VERSION_PLAN.md` for the full project definition.
