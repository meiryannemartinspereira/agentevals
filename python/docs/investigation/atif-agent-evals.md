ATIF → AgentEvals Investigation
Objective
Investigate LangChain AgentEvals issue #97:
[Feature Request] Add ATIF trajectory input support
Goal: understand the current trajectory model and find the smallest safe way to add ATIF support.
Current Status
Phase: Investigation
Implementation: Not started
Branch: main
Repository: langchain-ai/agentevals
What We Know
AgentEvals
AgentEvals evaluates trajectories using message-oriented data.
Main normalization:
_normalize_to_openai_messages_list()

Used by:
strict
subset
superset
unordered
match
llm

This suggests ATIF can potentially be converted before reaching the existing evaluators.
Tool Calls
AgentEvals already supports:
assistant
  └── tool_calls
        ├── id
        └── function
             ├── name
             └── arguments

tool
  └── tool_call_id

Tool calls are normalized through:
_normalize_tool_call()

ATIF
An ATIF step can contain:
message
reasoning
multiple tool_calls
observation
metrics

It can also contain metadata such as:
model
timestamp
cost
tokens
logprobs

Important difference:
ATIF
  └── execution step
       ├── message
       ├── reasoning
       ├── tool calls
       ├── observation
       └── metrics

AgentEvals
  └── messages
       ├── assistant
       └── tool

Initial Mapping
ATIF                         AgentEvals

step.message          --->   user/assistant message
step.tool_calls       --->   assistant.tool_calls
step.observation      --->   tool message
tool_call_id          --->   tool_call_id
function_name         --->   tool name
arguments             --->   tool arguments

Still unresolved:
reasoning
metrics
timestamp
model metadata
cost
logprobs
extra

Progress
Completed
- [x] Read issue #97
- [x] Inspect AgentEvals trajectory structure
- [x] Inspect utils.py
- [x] Inspect trajectory tests
- [x] Identify shared message normalization
- [x] Identify tool-call normalization
- [x] Obtain real ATIF example
- [x] Compare ATIF and AgentEvals structures
Next
- [ ] Understand tool_call_id ↔ tool result relationship
- [ ] Validate ATIF → AgentEvals mapping
- [ ] Inspect reference implementation
- [ ] Reproduce the current limitation
- [ ] Decide the smallest adapter design
Constraint
Prefer a lightweight adapter:
ATIF trajectory
      |
      v
atif_to_openai_messages(...)
      |
      v
existing AgentEvals evaluators

Avoid changing existing evaluator APIs unless necessary.
Working Principle
Understand
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

No implementation yet.
Checkpoint
🔎 Investigation
      ↓
ATIF structure understood
      ↓
AgentEvals normalization understood
      ↓
Tool-call support identified
      ↓
>>> NEXT: understand tool_call_id + observations <<<
      ↓
Validate mapping
      ↓
Reproduce
      ↓
Implement