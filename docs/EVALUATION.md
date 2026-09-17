# Evaluation Strategy

Agent evaluation should separate **hard correctness**, **trajectory quality**, and **final outcome quality**. Do not use an LLM judge for everything.

## Evaluation layers

### 1. Tool-call correctness

Prefer deterministic checks where possible.

Evaluate:

- correct tool selected
- arguments match schema
- arguments satisfy constraints
- required fields present
- prohibited actions not attempted
- retries are safe
- redundant calls are minimized

Example assertions:

```text
budget <= 120 → search_products.max_price <= 120
product_id passed to get_reviews() exists
add_to_cart() cannot execute before approval
```

### 2. Planning and trajectory quality

Planning rarely has one perfect gold trajectory. Use a rubric instead of exact-step matching.

Questions:

- Did the plan cover all necessary subtasks?
- Did the agent gather enough evidence before deciding?
- Did it investigate high-uncertainty choices?
- Did it avoid irrelevant work?
- Did it update the plan after new information?
- Did it stop when enough information was available?

Bootstrap this with human-reviewed examples. Then test whether an LLM judge can reproduce human judgments well enough to scale evaluation.

Recommended process:

```text
human examples
    ↓
define rubric
    ↓
label seed eval set
    ↓
LLM judge
    ↓
measure agreement with humans
    ↓
scale judge + continue human audits
```

### 3. Final-output quality

Evaluate:

- all hard constraints satisfied
- recommendations supported by tool evidence
- no invented product facts
- requested categories covered
- useful tradeoffs explained
- uncertainty surfaced when evidence is incomplete

Combine deterministic checks with rubric-based review.

### 4. System quality

Track:

- end-to-end latency
- model latency
- tool latency
- token usage
- model cost
- number of tool calls
- retries
- failure rate
- loop rate
- recovery rate

## V1 eval set

Create 20–30 tasks spanning:

- simple exact constraints
- ambiguous intent
- no results
- conflicting reviews
- missing metadata
- tool errors
- products near the budget boundary

Example:

> Find an ergonomic keyboard under $120 for all-day typing. Recommend three options and explain your preferred choice.

Expected checks:

```text
all recommended products <= $120
all product IDs exist
reviews were consulted before ergonomic claims
no unsupported product attributes
```

## V2 eval set

Add multi-category bundle tasks with:

- hard total budget
- required categories
- soft preferences
- incompatible constraints
- missing information
- changing availability

Evaluate both plan quality and final bundle validity.

## V3 eval set

Add action safety cases:

- approval granted
- approval denied
- tool timeout after successful execution
- retry after ambiguous failure
- invalid item
- permission failure

Key invariant:

**External state must never change outside the allowed approval and policy boundary.**

## V4 eval set

Test decomposition and orchestration:

- independently parallelizable subtasks
- tasks with dependencies that must not run in parallel
- one worker fails
- one worker returns low-quality evidence
- duplicate worker tasks
- aggregation conflicts

Compare V4 against a sequential single-agent baseline on quality, latency, and cost.

## Failure-driven regression

Every meaningful production-like failure should become a test case.

Example:

```text
Failure:
Agent interpreted a variant price as the canonical product price.

Fix:
Separate variant and offer schemas and add explicit price-source metadata.

Regression:
Add eval task ensuring variant and merchant offer prices are distinguished.
```

This creates a practical improvement loop instead of a static benchmark.
