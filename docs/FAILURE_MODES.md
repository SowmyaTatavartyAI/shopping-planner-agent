# Failure Modes and Debugging Playbook

The goal of this project is not only to make the agent work, but to learn how to debug it systematically.

## Debugging order

When behavior is wrong, inspect failures in this order:

1. **Tool contract** — Was the tool description or schema ambiguous?
2. **Input/state** — Did the model have the information it needed?
3. **Instructions/skill** — Was the behavioral policy clear enough?
4. **Planning** — Did the agent decompose or sequence the task poorly?
5. **Model judgment** — Was the decision itself bad despite correct context and tools?
6. **System/runtime** — Did latency, timeout, concurrency, or state persistence cause the failure?

This avoids treating every failure as "the model was wrong."

## 1. Wrong tool selected

Possible causes:

- overlapping tool descriptions
- vague names
- missing constraints in the tool schema
- relevant tool not present in context
- misleading prior tool result

Debug by comparing the available tool descriptions at the exact failing step.

## 2. Correct tool, wrong arguments

Possible causes:

- weak parameter descriptions
- values buried in conversation context
- bad constraint extraction
- schema accepts overly broad input

Mitigations:

- typed schemas
- enums where appropriate
- deterministic validation
- structured state
- reject-and-correct loops for invalid arguments

## 3. Repeated or looping calls

Symptoms:

- same tool called with same parameters
- search repeated without meaningful query change
- agent never reaches stopping condition

Mitigations:

- record tool-call history
- explicit stop criteria
- duplicate-call guard
- max-step limit
- surface progress in structured state

## 4. Insufficient research

The agent reaches a conclusion without gathering evidence necessary to support it.

Example:

It recommends a keyboard for all-day ergonomic use but never inspects ergonomic attributes or reviews.

Mitigations:

- skill/rubric defining decision-critical evidence
- uncertainty-aware research
- trajectory evals

## 5. Over-research

The agent keeps gathering information after the decision is already stable.

Measure:

- unnecessary tool calls
- marginal evidence gained per call
- cost and latency

Mitigation: define stopping conditions and teach the agent to research only when additional evidence could change the decision.

## 6. Lost global objective

Common in longer tasks.

Symptoms:

- one category consumes most of the budget
- required categories are forgotten
- early preferences disappear from later reasoning

Mitigations:

- explicit task state
- structured plan
- progress tracking
- replanning after major observations

## 7. Stale or conflicting state

Possible when conversation messages and application state disagree.

Mitigations:

- define one canonical state source
- version state mutations
- validate state transitions
- avoid relying on reconstructed natural-language history for hard constraints

## 8. Tool timeout

Differentiate:

- request definitely failed
- request may have executed but response was lost

For read-only tools, retry may be safe.

For actions, blind retry can be dangerous.

Use:

- idempotency keys
- state verification
- bounded retries
- backoff

## 9. Partial tool failure

Example: product search succeeds but review service is down.

The agent should not necessarily fail the whole task. It may:

- degrade gracefully
- state uncertainty
- use alternate evidence
- ask the user whether to proceed

The acceptable behavior depends on how important the missing evidence is.

## 10. Unsupported claims / hallucination

The model may infer product facts that were not returned by any tool.

Mitigations:

- provenance in evidence state
- require product claims to map to retrieved fields or sources
- deterministic post-checks where possible
- faithfulness evals

## 11. Prompt injection from tool output

Treat external text as data, not trusted instructions.

Examples:

- malicious review text
- merchant description containing instructions to the model
- retrieved page telling the agent to ignore system rules

Mitigations:

- strong instruction hierarchy
- delimit untrusted content
- limit action permissions
- separate reading from execution
- validate action intent in code

## 12. Budget violation

Never rely only on model arithmetic for a hard budget.

Use deterministic code to calculate:

```text
total = sum(selected_offer.price * quantity)
assert total <= budget
```

## 13. Product/offer/variant confusion

Commerce systems often distinguish:

- canonical product group
- variant/product
- merchant offer/SKU

A recommendation may be correct at the product level while price or availability belongs to an offer.

Use explicit IDs and schemas for each entity.

## 14. Duplicate consequential action

Example:

1. agent calls `add_to_cart()`
2. backend succeeds
3. response times out
4. agent retries
5. item is added twice

Mitigate with idempotency keys and verification before retry.

## 15. Worker failure in V4

When parallel workers are used, define how to handle:

- timeout
- empty result
- conflicting results
- one failed subtask
- duplicated work

The orchestrator should know which results are required vs optional and whether degraded completion is acceptable.

## Trace template

For every failure, capture:

```text
Task:
Expected behavior:
Observed behavior:
First incorrect step:
State at that step:
Available tools:
Tool/model action:
Observation returned:
Root cause hypothesis:
Fix:
Regression test added:
```

This turns debugging into a repeatable engineering process rather than prompt guessing.
