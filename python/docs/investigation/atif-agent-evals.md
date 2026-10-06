# ATIF → AgentEvals Investigation

## Objective

Investigate LangChain AgentEvals issue #97:

**[Feature Request] Add ATIF trajectory input support**

Goal: understand the problem, the current AgentEvals trajectory model, the ATIF format, and the safest way to introduce ATIF trajectory support without unnecessarily changing the existing evaluator APIs.

---

## Current Status

**Phase:** Investigation

**Implementation started:** No

**Current branch:** `main`

**Repository:** `langchain-ai/agentevals`

**Local path:**

```text
~/Documentos/agentevals
```

---

## What We Know

### AgentEvals

AgentEvals evaluates agent trajectories.

The current trajectory evaluators work primarily with:

- OpenAI-style messages
- LangChain `BaseMessage`

The existing trajectory implementation normalizes messages into an OpenAI-style representation before performing trajectory matching/evaluation.

Relevant directory:

```text
python/agentevals/trajectory/
```

Relevant file inspected:

```text
python/agentevals/trajectory/utils.py
```

The current implementation contains utilities for:

- converting messages to OpenAI-style messages
- normalizing message lists
- normalizing tool calls
- extracting tool calls
- comparing trajectories
- strict/subset/superset matching

---

## ATIF

ATIF (Agent Trajectory Interchange Format) represents an agent execution trajectory.

The real example inspected during the investigation is a multi-step stock-price task.

Simplified execution flow:

```text
USER
  |
  v
STEP 1
"What is the current trading price of Alphabet?"
  |
  v
AGENT
  |
  +-- reasoning
  |
  +-- tool call: financial_search(price)
  |
  +-- tool call: financial_search(volume)
  |
  +-- observations
  |
  v
STEP 3
FINAL AGENT RESPONSE
```

The ATIF example contains information such as:

- `schema_version`
- `session_id`
- agent metadata
- model name
- tool definitions
- notes
- final metrics
- steps
- timestamps
- step source
- messages
- reasoning content
- tool calls
- observations
- token metrics
- cost
- log probabilities
- completion token IDs
- additional metadata

---

## Important Observation

An ATIF step is not necessarily equivalent to one AgentEvals message.

For example, a single ATIF agent step can contain:

```text
message
reasoning_content
multiple tool_calls
observation
metrics
```

AgentEvals currently works with a message-oriented representation such as:

```text
assistant message
  |
  +-- tool_calls

tool message
  |
  +-- tool result
```

Therefore, the central investigation question is not simply:

> "How do we convert ATIF JSON to AgentEvals JSON?"

The more important question is:

> **Which ATIF information can be mapped to AgentEvals without losing information that matters for trajectory evaluation?**

---

## Initial Mapping Hypothesis

Current hypothesis:

```text
ATIF                         AgentEvals

step.message          --->   assistant/user message

step.tool_calls       --->   assistant.tool_calls

step.observation      --->   tool message

tool_call_id          --->   tool_call_id

function_name         --->   tool/function name

arguments             --->   tool arguments
```

However, the following fields do not currently have an obvious direct destination:

```text
reasoning_content
timestamp
model_name
reasoning_effort
metrics
cost
logprobs
completion_token_ids
extra
```

This mapping is still a hypothesis and has not yet been validated against the actual AgentEvals implementation.

---

## Existing AgentEvals Tests Inspected

File:

```text
python/tests/test_trajectory.py
```

The tests represent trajectories using OpenAI-style messages.

Example conceptual trajectory:

```text
user
  |
assistant + tool_call
  |
tool + result
  |
assistant + final response
```

The tests also cover trajectories containing multiple tool calls.

This is important because the ATIF example also contains multiple tool calls within a single agent step.

---

## Investigation So Far

### Completed

- [x] Clone AgentEvals repository
- [x] Verify repository state
- [x] Inspect Python project structure
- [x] Inspect trajectory implementation
- [x] Inspect trajectory utility functions
- [x] Inspect trajectory tests
- [x] Read issue #97
- [x] Obtain a real ATIF trajectory example
- [x] Compare the conceptual structure of ATIF and AgentEvals

### Not Started

- [ ] Inspect every relevant AgentEvals message conversion path
- [ ] Determine exactly what information AgentEvals preserves
- [ ] Determine exactly what information would be lost by an ATIF adapter
- [ ] Find or inspect the proposed/reference implementation mentioned by issue #97
- [ ] Reproduce the current limitation locally
- [ ] Define the smallest acceptable adapter design
- [ ] Implement
- [ ] Add tests
- [ ] Update documentation
- [ ] Open pull request

---

## Questions to Answer

1. How does AgentEvals currently represent a trajectory internally?

2. How are multiple ATIF tool calls represented after conversion?

3. How should ATIF observations map to AgentEvals tool messages?

4. What should happen to `reasoning_content`?

5. Should metadata such as timestamps and metrics be preserved?

6. Does the existing AgentEvals data model have a safe place for ATIF-specific metadata?

7. What information can be discarded without affecting trajectory evaluation?

8. Does the proposed adapter need to support the complete ATIF specification or only the fields required by AgentEvals?

9. Can the adapter remain independent from the ATIF schema implementation?

10. Can existing AgentEvals evaluator APIs remain unchanged?

---

## Important Constraint

The investigation should avoid unnecessary changes to AgentEvals.

The issue proposal suggests a lightweight adapter approach:

```text
ATIF trajectory
      |
      v
atif_to_openai_messages(...)
      |
      v
existing AgentEvals evaluators
```

The preferred direction is to reuse the existing evaluator infrastructure rather than introduce a new trajectory evaluation model.

---

## Working Principle

Before implementing anything:

```text
Understand
   ↓
Observe
   ↓
Reproduce
   ↓
Map
   ↓
Decide
   ↓
Implement
   ↓
Test
```

No implementation should begin until the current behavior and the required mapping are understood.

---

## Next Step

Inspect where `ChatCompletionMessage` and message normalization are used throughout:

```text
python/agentevals/trajectory/
python/tests/
```

Command planned:

```bash
grep -R "ChatCompletionMessage" agentevals/trajectory tests -n
```

The result should help identify the exact boundaries where an ATIF adapter would connect to the existing AgentEvals implementation.

---

## Checkpoint

Last known state:

> We have a real ATIF trajectory example and have identified the key conceptual difference: ATIF represents an execution step containing multiple kinds of information, while AgentEvals currently evaluates a normalized message-oriented trajectory.

**Current position:**

```text
🔎 Investigation
      ↓
ATIF structure understood
      ↓
AgentEvals structure partially understood
      ↓
>>> NEXT: inspect exact message boundaries <<<
      ↓
Mapping
      ↓
Reproduction
      ↓
Implementation
```