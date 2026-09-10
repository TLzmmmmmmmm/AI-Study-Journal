# LangGraph Fundamentals Notes

> Week 3 · Day 5  
> Goal: Understand how LangGraph represents Agent control flow using explicit state, nodes, edges, loops, and tool execution.

---

# 1. Why LangGraph Exists

A simple LLM application often follows a fixed pipeline:

```text
Input
  ↓
Retrieve
  ↓
Prompt
  ↓
LLM
  ↓
Output
```

This works well when execution order is predetermined.

Agent systems are different:

```text
User Goal
   ↓
LLM Decision
   ↓
Need Tool?
 ├─ Yes → Tool → LLM
 └─ No  → END
```

Execution may contain:

- branches
- loops
- tool calls
- retries
- persistent state
- human approval

LangGraph provides explicit infrastructure for these stateful workflows.

> **LangGraph models Agent execution as state moving through a graph of executable nodes.**

---

## 1.1 LangChain vs LangGraph

### LangChain

Provides reusable LLM application components such as:

- models
- prompts
- tools
- retrievers
- agent abstractions

Best suited for quickly assembling common LLM applications.

### LangGraph

Focuses on:

- state
- control flow
- branching
- loops
- persistence
- durable execution
- human-in-the-loop

```text
LangChain
= LLM application components

LangGraph
= stateful workflow orchestration
```

They are complementary rather than mutually exclusive.

---

# 2. Core Mental Model

LangGraph has three fundamental concepts:

```text
State
+
Node
+
Edge
```

A graph executes roughly as:

```text
START
  ↓
Node
  ↓
State Update
  ↓
Edge
  ↓
Next Node
  ↓
...
  ↓
END
```

---

## 2.1 State

> **State is the shared data contract of the graph.**

Example:

```text
AgentState

messages
retrieval_result
tool_result
route
sources
iteration_count
```

Nodes primarily communicate through State rather than directly passing arbitrary parameters to one another.

Traditional Python:

```text
A() → result → B(result)
```

LangGraph:

```text
Node A
  ↓
Update State
  ↓
Node B reads State
```

State answers:

> **What does the workflow currently know?**

---

## 2.2 Node

A Node is an executable unit of work.

It may:

- call an LLM
- execute a tool
- perform retrieval
- validate output
- transform data
- make a classification

General pattern:

```text
State
  ↓
Node
  ↓
Work
  ↓
Partial State Update
```

A Node normally returns only the fields it wants to update.

> **Node = computation or action.**

---

## 2.3 Edge

Edges define control flow between Nodes.

### Normal Edge

Fixed execution:

```text
A → B → C
```

### Conditional Edge

Dynamic execution:

```text
       → B
A → decision
       → C
       → END
```

> **Nodes perform work; Edges determine what executes next.**

---

# 3. State Updates and Reducers

When a Node returns:

```text
{"field": new_value}
```

LangGraph must determine how this update affects the existing State.

Without a Reducer:

```text
old_value
   ↓
new_value replaces it
```

With a Reducer:

```text
old_value
+
new_update
   ↓
Reducer
   ↓
merged_value
```

Therefore:

> **State defines what data exists; Reducers define how that data evolves.**

---

## 3.1 Overwrite vs Accumulate

Current-value fields usually overwrite:

```text
current_route
current_answer
current_product
```

Historical or accumulated fields often use reducers:

```text
messages
events
history
scores
```

Mental rule:

```text
Current value
→ overwrite

History / collection
→ reducer
```

---

## 3.2 Reducer Is Per Field

Different fields can use different update rules:

```text
State

route
→ overwrite

iteration_count
→ addition

messages
→ message-aware merge
```

A Reducer can conceptually be any function:

```text
Reducer(old, new) → merged
```

Examples:

- numeric addition
- list concatenation
- deduplication
- dictionary merge
- custom aggregation

---

# 4. Conditional Routing

Conditional Edges convert decisions into executable control flow.

```text
Source Node
    ↓
Routing Function reads State
    ↓
Route Decision
    ↓
Target Node
```

The decision may come from:

- deterministic rules
- classifier models
- LLM decisions
- tool results
- validation results
- current State

Important:

> **Conditional routing is not itself intelligence.**

For example:

```text
Rule
↓
Conditional Edge
```

is still a deterministic workflow.

An LLM may instead produce:

```text
next_action = search_products
```

and the graph converts that decision into:

```text
LLM Node
   ↓
search_products Node
```

---

# 5. Loops

Unlike a simple pipeline, a graph can return to an earlier Node.

```text
Generate
   ↓
Check
   ↓
Good enough?
├─ Yes → END
└─ No  → Generate
```

This enables iterative workflows.

Agent systems commonly use:

```text
LLM
 ↓
Tool
 ↓
LLM
 ↓
Tool
 ↓
...
 ↓
END
```

---

## 5.1 Termination Conditions

Every loop needs an explicit stopping rule.

Examples:

- task completed
- no tool call requested
- quality threshold reached
- maximum iterations reached
- unsafe to continue

Without termination guards:

```text
LLM → Tool → LLM → Tool → ...
```

may continue indefinitely.

Consequences include:

- higher latency
- higher token usage
- higher cost
- repeated actions
- unstable behavior

> **Every Agent loop requires a termination guard.**

---

## 5.2 Loop ≠ Agent

A workflow such as:

```text
Generate → Check → Generate
```

contains a loop but is not necessarily an Agent.

An Agent additionally involves autonomous action selection:

```text
Current State
     ↓
LLM Decision
     ↓
Tool A / Tool B / Ask User / END
```

---

# 6. MessagesState

Agent workflows often need to preserve the complete interaction trajectory.

Example:

```text
HumanMessage
↓
AIMessage(tool call)
↓
ToolMessage(result)
↓
AIMessage(tool call)
↓
ToolMessage(result)
↓
AIMessage(final answer)
```

`MessagesState` provides a predefined State structure for this message history.

Conceptually:

```text
messages
→ list of structured messages
```

Typical message roles include:

```text
SystemMessage
HumanMessage
AIMessage
ToolMessage
```

These correspond roughly to chat API roles:

```text
system
user
assistant
tool
```

---

## 6.1 `add_messages`

`add_messages` is a message-aware Reducer.

Generic list addition:

```text
old + new
→ append
```

Message-aware merge:

```text
existing messages
+
new messages
↓
merge while respecting message identity
```

It allows Agent history to grow without overwriting previous messages.

---

## 6.2 Messages Are Agent Trajectory

Messages do not only store conversation text.

They can also represent:

```text
User Request

AI Tool Call

Tool Observation

AI Tool Call

Tool Observation

Final Answer
```

This trajectory allows every new LLM invocation to see the results of previous actions.

---

## 6.3 MessagesState Is Not Persistent Memory

Important distinction:

```text
MessagesState
= how conversation state is represented

Checkpointer
= how graph state persists across runs

Long-term Memory
= durable information reused across threads
```

`MessagesState` alone does not make information survive process restarts or future sessions.

---

# 7. Tool Calling

Tools expose external capabilities to the Agent.

Examples:

```text
search_products
get_product_details
retrieve_knowledge
query_database
create_ticket
```

A good Tool has a clear contract:

```text
Tool Contract

Name
Description
Input Schema
Output Semantics
Error Behavior
```

---

## 7.1 Tool Schema vs Tool Execution

The LLM does not directly execute Python code.

Instead:

```text
Python Tool
   ↓
Tool Schema
   ↓
LLM sees available capability
```

The LLM may return:

```text
Tool Call:
search_products(...)
```

Then the backend executes it.

Full runtime:

```text
LLM
 ↓
Tool Call Request
 ↓
Backend Validation
 ↓
Python Tool Execution
 ↓
Tool Result
 ↓
State / ToolMessage
 ↓
LLM again
```

> **The LLM chooses an action; the backend owns validation and execution.**

---

## 7.2 Tool Design Principles

Prefer:

- single responsibility
- explicit schemas
- structured outputs
- deterministic behavior
- explicit error states
- least privilege

Avoid:

```text
do_everything()
```

Prefer:

```text
search_products()
get_product_details()
get_contact_info()
```

For high-impact actions:

```text
send_email
issue_refund
create_order
delete_data
```

additional validation or human approval may be required.

---

# 8. ReAct Agent

ReAct = Reasoning + Acting.

Conceptual loop:

```text
Reason
  ↓
Action
  ↓
Observation
  ↓
Reason
  ↓
...
```

In a modern tool-calling implementation:

```text
LLM Node
   ↓
Tool Call?
├─ No  → END
│
└─ Yes
     ↓
   Tool Node
     ↓
   Tool Result
     ↓
   MessagesState
     ↓
   LLM Node
```

---

## 8.1 Thought

`Thought` is a conceptual reasoning stage.

It does not need to appear as:

```text
Thought: ...
```

in the model output.

Instead:

```text
Messages
   ↓
LLM reasoning
   ↓
Decision
```

The externally observable result may simply be:

```text
tool_calls = [...]
```

or:

```text
final answer
```

Therefore:

```text
Thought
→ internal reasoning

Action
→ structured tool call

Observation
→ tool result
```

The system should observe decisions and actions rather than depend on exposing full internal reasoning.

---

## 8.2 Model Node

The Model Node receives current messages and asks the LLM for the next decision.

Possible outputs:

```text
AIMessage(final answer)
```

or:

```text
AIMessage(tool call)
```

The Model Node is not the whole Agent.

```text
Agent
=
Model Node
+
Tools
+
State
+
Routing
+
Loop
```

---

## 8.3 Tool Node

A Tool Node typically:

```text
Read Tool Call
↓
Find Tool
↓
Validate Arguments
↓
Execute Tool
↓
Create Tool Result
↓
Update State
```

The graph then routes execution back to the LLM.

---

# 9. RAG and LangGraph

RAG and LangGraph solve different problems.

```text
RAG
= knowledge retrieval

Agent
= decision and action

LangGraph
= stateful Agent/workflow orchestration
```

They can be combined:

```text
              Agent Graph

                   |
        +----------+----------+
        |                     |
        v                     v
     RAG Node              Tool Node
        |                     |
    Retriever             External API
        |
   Knowledge Base
```

Example:

```text
User Question
↓
Agent decides knowledge is required
↓
RAG Retrieval
↓
Relevant Evidence
↓
State Update
↓
LLM
↓
Grounded Answer
```

> **RAG provides knowledge; LangGraph controls when and how that capability is used.**

---

# 10. Persistence and Memory

Agent workflows may span multiple interactions.

LangGraph uses checkpointing to persist graph State.

Conceptually:

```text
State 0
  ↓
Node A
  ↓
State 1 ── Checkpoint
  ↓
Node B
  ↓
State 2 ── Checkpoint
```

A thread identifies a sequence of related checkpoints.

```text
thread_1
→ conversation / workflow A

thread_2
→ conversation / workflow B
```

A user may have multiple threads.

Therefore:

```text
thread_id ≠ necessarily user_id
```

---

## 10.1 Short-Term vs Long-Term Memory

### Checkpointer

Stores thread-scoped graph state.

Useful for:

- conversation continuity
- resume after interruption
- fault recovery
- human approval workflows

### Long-Term Store

Stores durable information across threads.

Examples:

- user preferences
- stable facts
- historical experiences
- reusable memories

Mental model:

```text
Checkpointer
= where is this workflow now?

Long-term Memory
= what should the system remember beyond this workflow?
```

---

# 11. Human-in-the-Loop

Some Agent actions should not execute autonomously.

Example:

```text
Agent proposes action
        ↓
      PAUSE
        ↓
   Human Review
    /        \
Approve     Reject
  |            |
Action        END
```

Human-in-the-Loop is useful for:

- financial actions
- destructive operations
- external communication
- permission-sensitive actions
- uncertain decisions

The graph must preserve State while paused so execution can later resume.

> **The greater the consequence of an action, the less autonomy the Agent should receive.**

---

# 12. Subgraphs

Complex workflows can be modularized into Subgraphs.

```text
Main Graph

Router
  ↓
[Product Subgraph]
  ↓
Generator
```

The Subgraph may internally contain:

```text
Search
 ↓
Filter
 ↓
Resolve
 ↓
Validate
```

From the parent graph's perspective, this entire workflow can behave like one unit.

> **Subgraph = reusable Graph encapsulated inside another Graph.**

Benefits:

- modularity
- clearer architecture
- reusable workflows
- isolated domain logic

Do not introduce Subgraphs before system complexity justifies them.

---

# 13. Multi-Agent Systems

A Multi-Agent system separates decision-making responsibilities.

Supervisor pattern:

```text
                Supervisor
               /    |     \
              /     |      \
             v      v       v
          Agent A Agent B Agent C
```

Supervisor:

```text
decides WHO handles the task
```

Specialist Agent:

```text
decides HOW to solve its assigned task
```

Each specialist may itself be a Subgraph.

---

## 13.1 Tool vs Agent

Important distinction:

```text
Tool
= executes a capability

Agent
= owns a decision loop
```

A Python function that directly handles one task is not automatically another Agent.

---

## 13.2 When Multi-Agent Helps

Useful when specialization provides:

- domain isolation
- context isolation
- tool isolation
- permission isolation
- independent workflows

Costs:

- more LLM calls
- more latency
- more state complexity
- harder debugging
- more failure points

> **Prefer one Agent with well-designed tools until multiple Agents provide a clear architectural benefit.**

---

# 14. Streaming

Two different types of streaming should be distinguished.

## Model Streaming

```text
LLM
↓
Token / Chunk
↓
User Interface
```

Purpose:

> Display generated output incrementally.

---

## Graph Streaming

```text
Graph
↓
Node Event
↓
State Update
↓
Tool Event
↓
Next Node
```

Purpose:

> Observe workflow execution incrementally.

These are separate from the HTTP transport protocol.

For example:

```text
LangGraph Internal Stream
        ↓
FastAPI
        ↓
NDJSON
        ↓
Frontend
```

LangGraph does not need to replace the application's existing HTTP streaming design.

---

# 15. Observability and Debugging

Agent failures often do not produce exceptions.

Example:

```text
LLM chooses wrong Tool
↓
Tool executes successfully
↓
Valid result returned
↓
LLM continues on wrong path
↓
Incorrect final answer
```

Therefore Agent observability should capture:

```text
Input
↓
Route Decision
↓
LLM Decision
↓
Tool Call
↓
Tool Result
↓
State Transition
↓
Final Answer
```

Useful debugging views include:

```text
Graph Structure
+
Runtime Trace
```

Goal:

> **Agent failures should be attributable to a specific decision, tool, state transition, or generation step.**

---

# 16. Mapping to a Python Agent Backend

Without LangGraph, orchestration may look like:

```text
FastAPI
  ↓
Python Router
  ↓
Retriever / Tool
  ↓
Agent Loop
  ↓
LLM
```

With LangGraph:

```text
FastAPI
  ↓
Compiled Graph
  ↓
State
  ↓
Nodes + Edges + Loops
```

Existing capabilities remain reusable:

```text
Retriever
Tool Domain Layer
Context Builder
Prompt Builder
LLM Client
Citation Collector
```

LangGraph primarily replaces or organizes the **control-flow layer**.

It does not inherently replace:

- FastAPI
- LLM provider
- retriever
- vector index
- knowledge base
- HTTP streaming protocol

---

## 16.1 Example Agent Graph

```text
                     START
                       ↓
                   LLM Node
                       ↓
                 Need Action?
                 /          \
               No            Yes
               |              |
              END        Tool Routing
                           /   |   \
                          /    |    \
                         v     v     v
                      Search Detail Contact
                         \     |     /
                          \    |    /
                           Tool Result
                               ↓
                           LLM Node
```

State may contain:

```text
messages
retrieval_result
sources
route
tool_result
iteration_count
```

Different fields should remain structured instead of placing all application data inside `messages`.

---

# 17. When to Use LangGraph

LangGraph is useful when the application requires several of:

- dynamic branching
- iterative Agent loops
- multiple tools
- durable State
- resumable workflows
- human approval
- complex error recovery
- explicit orchestration
- multi-Agent coordination

---

## 17.1 When Plain Python May Be Better

For a simple deterministic pipeline:

```text
Input
↓
Retrieve
↓
Generate
↓
Output
```

normal Python may be:

- simpler
- easier to debug
- easier to test
- less abstract

Do not add LangGraph only because the application contains an LLM.

Use it when explicit graph-based orchestration solves real control-flow complexity.

---

# Core Mental Model

```text
State
= what the workflow knows

Node
= what the workflow does

Reducer
= how State updates are merged

Edge
= where execution goes next

Conditional Edge
= dynamic routing

Loop
= repeated execution

MessagesState
= Agent interaction trajectory

Tool
= external capability

ReAct
= LLM → Action → Observation loop

Checkpointer
= thread-scoped State persistence

Long-Term Memory
= durable information across threads

Subgraph
= modular workflow

Human-in-the-Loop
= controlled human intervention
```

---

# Core Summary

> **LangGraph is a stateful orchestration framework for building controllable Agent and workflow systems.**

> **State stores execution context, Nodes perform work, Reducers control State updates, and Edges define execution flow.**

> **A ReAct Agent is a loop where the LLM selects an action, the backend executes it, the observation is added to State, and the LLM decides again until the task ends.**

> **RAG supplies knowledge, Tools supply capabilities, the LLM supplies decisions, and LangGraph supplies the control infrastructure that connects them.**

> **LangGraph becomes valuable when Agent behavior requires branching, loops, persistence, human control, or complex orchestration; simple deterministic pipelines may remain better as plain Python.**
