# ATIF → AgentEvals Investigation

## Objective

Investigate LangChain AgentEvals issue #97:
**[Feature Request] Add ATIF trajectory input support**

Goal: understand the current trajectory model and find the smallest safe way to add ATIF support.

## Current Status

**Phase:** Investigation  
**Implementation:** Not started  
**Branch:** `main`  
**Repository:** `langchain-ai/agentevals`

## What We Know

### AgentEvals

AgentEvals evaluates trajectories using message-oriented data.

Main normalization:

`_normalize_to_openai_messages_list()`

Used by strict, subset, superset, unordered, match and llm evaluators.

### AgentEvals Tool Calls

AgentEvals represents tool interactions as:

```text
assistant
└── tool_calls
    ├── id
    └── function
        ├── name
        └── arguments

tool
└── tool_call_id

Tool calls are normalized through _normalize_tool_call().
The trajectory matching logic compares tool names and arguments according to the configured match mode.
ATIF
An ATIF step can contain:
message
reasoning
tool_calls
observation
metrics
metadata

A tool call contains:
tool_call_id
function_name
arguments

An observation result can contain:
source_call_id
content

source_call_id links the observation to the corresponding tool_call_id.
Initial Mapping
AgentEvals                         ATIF

tool_calls[].id              ←→   tool_calls[].tool_call_id
function.name                ←→   function_name
function.arguments           ←→   arguments
tool.tool_call_id             ←→   observation.results[].source_call_id
tool.content                  ←→   observation.results[].content

Important Structural Difference
ATIF groups multiple pieces of an agent interaction inside one step:
step
├── message
├── tool_calls
├── observation
├── reasoning
└── metrics

AgentEvals expects a sequence of messages:
user
assistant + tool_calls
tool
assistant

Therefore, the adapter is not just a field rename.
It needs to transform an ATIF step into one or more AgentEvals messages while preserving the relationship between tool calls and observations.
Unresolved
Still need to determine:
- when step.message becomes user vs assistant
- how a step containing both message and tool_calls should be represented
- how multiple tool calls and observations are ordered
- what happens to reasoning, metrics, timestamp and other ATIF metadata
- exact behavior expected by the reference implementation
Progress
Completed:
- Read issue #97
- Inspected AgentEvals trajectory structure
- Inspected shared normalization
- Inspected tool-call normalization
- Inspected trajectory tests
- Confirmed tool-call matching behavior
- Recovered and inspected the ATIF schema/RFC
- Confirmed tool_call_id ↔ source_call_id relationship
- Compared ATIF and AgentEvals structures
- Identified that the main problem is structural transformation
Next Step
Understand the exact transformation:
ATIF step → AgentEvals message sequence
Before implementation, inspect the reference implementation and validate the mapping with concrete examples.
Working Principle
Understand
→ Reproduce
→ Map
→ Decide
→ Implement
→ Test

No implementation yet.