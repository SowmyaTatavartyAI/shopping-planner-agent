# Version Scope and Implementation Plan

This project is intentionally built as one evolving system. Each version introduces a new agent architecture concept only when the use case or failure mode justifies it.

## V1 — Single-Agent Shopping Researcher

### Goal

Handle a narrow shopping task end-to-end with one Claude agent and a small set of tools.

Example:

> Find three ergonomic keyboards under $120 and recommend one for someone who types all day.

### Architecture

```text
User
 ↓
Claude
 ↕
Tool loop
 ├── search_products()
 ├── get_product_details()
 ├── get_reviews()
 └── get_offers()
 ↓
Answer
```

### What to implement

Tools:

```text
search_products(query, category, min_price, max_price, filters)
get_product_details(product_id)
get_reviews(product_id, limit, aspect)
get_offers(product_group_id)
```

Each tool should have:

- precise description
- typed input schema
- typed output
- validation
- timeout handling
- explicit error result
- logging and tracing

Suggested structured state:

```text
ShoppingState
    user_request
    extracted_constraints
    products_seen
    evidence_collected
    current_selection
    tool_history
```

### Techniques tested

- tool-use loop
- tool schema design
- system instructions
- structured outputs
- context management
- stopping conditions
- retries
- tracing
- deterministic validation

### Evals

- tool selection accuracy
- argument accuracy
- hard-constraint satisfaction
- unsupported-claim rate
- redundant tool calls
- task completion

### Deliberate failures

- ambiguous tool descriptions
- empty search result
- review timeout
- overlapping tool responsibilities
- malformed tool result
- conflicting metadata and review evidence

---

## V2 — Multi-Step Shopping Planner

### Goal

Handle a shopping mission that requires decomposition, cross-category tradeoffs, state tracking, and replanning.

Example:

> Build my home office under $800. I need a monitor, keyboard, mouse, webcam, and headphones. Prioritize ergonomics and video calls.

### Architecture

```text
                      ┌── product search
                      ├── reviews
User → Claude/Planner ├── price history
          ↕           ├── offers
      Task State      ├── variants
          ↕           └── knowledge retrieval
       Replanning
          ↓
       Bundle
```

### New tools

```text
get_price_history(product_id)
get_variants(product_group_id)
retrieve_buying_guide(query)
```

### New capabilities

- explicit task decomposition
- structured plan representation
- progress tracking
- budget allocation
- replanning after observations
- retrieval / RAG
- procedural skills

### Skills

Example skill folders:

```text
skills/
    product_research/
    bundle_planning/
    review_analysis/
```

A tool gives the model an ability. A skill gives the model procedural guidance for how to use abilities effectively.

### Techniques tested

- implicit vs explicit planning
- plan → execute → observe → replan
- state vs conversational context
- retrieval grounding
- skill selection
- long-horizon context management
- uncertainty-driven research

### Evals

- did the plan cover all required categories?
- did the agent gather evidence for uncertain decisions?
- did it avoid unnecessary research?
- did it replan after discovering new constraints?
- did the final bundle satisfy budget and category requirements?
- does the final recommendation match the gathered evidence?

---

## V3 — Action-Taking Commerce Agent

### Goal

Move from recommendation-only behavior to consequential actions while preserving user control and system safety.

### New tools

```text
get_cart()
add_to_cart(product_id, quantity, idempotency_key)
remove_from_cart(product_id)
```

### Architecture

```text
Agent recommendation
        ↓
Proposed action
        ↓
Permission / policy layer
        ↓
User confirmation
        ↓
Execution
        ↓
Verify resulting state
```

### Techniques tested

- human-in-the-loop
- permission boundaries
- reversible vs consequential actions
- idempotency
- retry safety
- duplicate execution prevention
- authorization
- post-action verification
- prompt injection from tool output
- untrusted external data
- ambiguous failure recovery

### Example production failure

`add_to_cart()` executes successfully, but the HTTP response times out.

The agent must not blindly retry. It should verify cart state first or use an idempotency key.

### Evals

- action attempted only after required approval
- no duplicate action under retry
- correct recovery after timeout
- permission checks enforced
- final external state matches intended state

---

## V4 — Orchestration and Parallel Workers

### Goal

Handle complex missions whose subtasks can be decomposed and researched independently.

Example:

> Plan a birthday party for 15 six-year-olds with a $500 budget. I need decorations, favors, games, food-serving supplies, and a backup indoor activity.

### Architecture

```text
                       PLANNER
                          ↓
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
 Decorations research  Games research   Favors research
          ↓               ↓                ↓
          └───────────────┼────────────────┘
                          ↓
                    Bundle synthesis
                          ↓
                  Constraint verifier
```

### Worker rule

A worker does not have to be an agent.

Use deterministic code or a direct API call when the operation is known and reliable. Use an agent worker only when the subtask itself requires an open-ended reasoning loop.

### Techniques tested

- planner/executor architecture
- orchestrator-worker pattern
- parallel execution
- deterministic worker vs agent worker
- context isolation
- delegation
- aggregation
- concurrency
- failure isolation
- latency/cost tradeoffs

### Evals

- decomposition quality
- independence of subtasks
- correct aggregation
- worker failure recovery
- latency vs sequential baseline
- cost vs single-agent baseline
- no duplicated or conflicting work

---

## Suggested build order

| Step | Deliverable | Main concept |
|---|---|---|
| 1 | Mock commerce backend | deterministic system boundary |
| 2 | Four clean tools | tool schemas and descriptions |
| 3 | Single Claude agent | agent loop |
| 4 | Structured task state | state vs context |
| 5 | Trace viewer/log | observability |
| 6 | 20–30 eval tasks | evaluation harness |
| 7 | Complex bundle task | planning |
| 8 | RAG knowledge source | retrieval and grounding |
| 9 | Skills | procedural context |
| 10 | Price/offers/variants | richer commerce reasoning |
| 11 | Cart actions | HITL and execution safety |
| 12 | Fault injection | production debugging |
| 13 | Parallel workers | orchestration |
| 14 | Performance pass | latency, cost, reliability |
