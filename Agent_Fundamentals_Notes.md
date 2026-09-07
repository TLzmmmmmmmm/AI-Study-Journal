# AI Agent Fundamentals Notes

## 1. What is AI Agent

> **An AI Agent is a system that uses an LLM as a reasoning engine and combines tools, memory, and planning to autonomously complete tasks through interaction with an environment.**

Traditional LLM:

```
Input → LLM → Output
```

LLM mainly performs language generation.

Agent:

```
Goal
 |
 v
LLM Reasoning
 |
 +---- Tool
 |
 +---- Memory
 |
 +---- Planning
 |
 v
Action → Observation → Next Action
```

Core difference:

**LLM generates answers.**

**Agent selects actions to achieve goals.**

---

# 2. LLM vs Agent

## Traditional Chatbot

Characteristics:

- Responds based on current context
- Mainly produces text output
- Cannot directly interact with external systems

Limitations:

- No autonomous action
- No external information access
- Limited task execution ability


## AI Agent

Characteristics:

- Goal-oriented
- Multi-step reasoning
- Tool usage
- State management


Example:

User:

> Find a communication device suitable for a hotel.

Chatbot:

```
Recommend several products.
```

Agent:

```
1. Understand requirements
2. Search product knowledge
3. Compare specifications
4. Generate recommendation
```

---

# 3. Agent Architecture

An Agent usually contains four components:

```
              Agent

        +---------------+
        |      LLM      |
        |  Reasoning    |
        +---------------+

        +---------------+
        |    Tools      |
        |   Actions     |
        +---------------+

        +---------------+
        |    Memory     |
        | Information   |
        +---------------+

        +---------------+
        |   Planning    |
        |  Strategy     |
        +---------------+
```

---

## 3.1 LLM — Brain

LLM provides:

- reasoning ability
- language understanding
- decision generation

However:

> LLM itself cannot access external systems or execute real-world actions.

Tools and memory extend its capability.

---

## 3.2 Tools — Hands

Tools connect Agent with external systems.

Examples:

- Database
- Search engine
- Calculator
- API
- CRM system


Tool workflow:

```
LLM
 |
Tool Request
 |
Validation
 |
Execution
 |
Result
 |
LLM continues reasoning
```

A reliable tool requires:

- clear responsibility
- defined input/output format
- error handling

---

## 3.3 Memory

Memory allows Agent to maintain information across interactions.

### Short-term Memory

Current conversation state:

- recent messages
- current task
- temporary information

Usually stored in context window.


### Long-term Memory

Persistent information:

- user profile
- preferences
- historical records

Usually stored in databases or vector stores.


### Episodic Memory

Past experiences:

- previous successful workflows
- historical solutions
- learned patterns

---

# 4. Agent Loop

The core mechanism of Agent is continuous decision making.

A common pattern is ReAct:

```
Reason
  |
  v
Act
  |
  v
Observe
  |
  v
Reason
  |
 Repeat
```

Example:

```
User:
Recommend a product for mining.


Reason:
Need product information.

Action:
Search product database.

Observation:
Receive product list.

Reason:
Need compare specifications.

Action:
Retrieve details.

Final:
Generate recommendation.
```

Agent terminates when:

- task is completed
- goal is achieved
- continuing is unsafe or unnecessary

---

# 5. Tool Calling

Tool calling is the connection between LLM reasoning and software execution.

LLM does not directly execute code.

Instead:

```
LLM
 |
Generate Tool Call
 |
Backend Executes Tool
 |
Return Result
 |
LLM Produces Next Step
```

Good tool design:

## Single Responsibility

Good:

```
search_products()

get_product_details()

create_customer_lead()
```

Bad:

```
do_everything()
```


## Stable Contract

Tool should define:

- input schema
- output schema
- failure behavior


Agent reliability depends heavily on tool reliability.

---

# 6. Planning

Planning allows Agent to solve complex goals by decomposing them into steps.

Without planning:

```
Goal → Answer
```

With planning:

```
Goal

↓

Step 1

↓

Step 2

↓

Step 3

↓

Result
```

---

## ReAct

Dynamic planning:

```
Think → Act → Observe → Think
```

Suitable for:

- uncertain environments
- exploration tasks


## Plan-and-Execute

First generate a plan:

```
Plan:

1. Collect information
2. Analyze data
3. Produce result
```

Then execute.

Suitable for:

- structured workflows
- business processes

---

# 7. Multi-Agent System

Multi-Agent means multiple specialized Agents cooperate.

Example:

```
Manager Agent

    |
    +---- Research Agent
    |
    +---- Data Agent
    |
    +---- Writing Agent
```

Advantages:

- specialization
- parallel execution
- complex task handling


Limitations:

- higher cost
- higher latency
- coordination complexity


Important:

> More Agents do not always mean better performance.

A single Agent with strong tools can outperform unnecessary multi-agent designs.

---

# 8. Agent Applications

## Customer Support Agent

Capabilities:

- answer customer questions
- retrieve knowledge
- search products
- create service requests


Architecture:

```
Agent

+ Retrieval Tool
+ Product Tool
+ CRM Tool
+ Human Escalation
```

---

## Coding Agent

Capabilities:

- understand codebase
- modify files
- run tests
- debug problems


---

## Research Agent

Capabilities:

- search information
- summarize sources
- compare findings


---

## Data Analysis Agent

Capabilities:

- query databases
- analyze datasets
- generate reports

---

# 9. Agent and RAG Relationship

> **RAG provides knowledge. Agent provides decision and action.**

RAG:

```
Question
 |
Retriever
 |
Relevant Context
 |
LLM
 |
Answer
```

Purpose:

- provide external knowledge
- reduce hallucination
- ground responses in sources


Agent:

```
Goal

↓

Reason

↓

Choose Action

↓

Execute Tool

↓

Observe Result

↓

Complete Task
```

Purpose:

- complete multi-step workflows


---

## Combining Agent + RAG

Production AI systems often use:

```
             Agent

               |
      +--------+--------+

      RAG Tool       Other Tools

      Knowledge      Database
      Retrieval      API
```

Example:

Customer:

> Which device is suitable for a hotel?


Agent:

1. Understand requirements
2. Call RAG retrieval tool
3. Retrieve product information
4. Compare options
5. Generate recommendation


Relationship:

**RAG is a capability.**

**Agent is the orchestration layer.**

---

# 10. Agent Evaluation

Agent evaluation focuses on whether the system completes tasks reliably.

## Task Success

Did Agent achieve the user's goal?


## Tool Selection Accuracy

Did Agent choose the correct tool?

Example:

Correct:

```
Product question → search_products()
```

Incorrect:

```
Product question → contact_support()
```


## Planning Quality

Evaluate:

- unnecessary steps
- logical reasoning
- failure recovery


## Efficiency

Measure:

- token usage
- number of tool calls
- latency
- cost

---

# 11. Production Considerations

## Reliability

Common failures:

- wrong tool selection
- infinite loops
- incorrect planning
- hallucinated actions


Solutions:

- tool validation
- iteration limits
- fallback strategies
- human approval


---

## Safety

Important controls:

- permission management
- sensitive action confirmation
- data protection


---

## Observability

Production Agent should track:

```
User Goal

↓

Agent Decision

↓

Tool Calls

↓

Results

↓

Final Response
```

Tracing is required for debugging and optimization.

---

# Core Summary

> **An AI Agent is not simply a smarter chatbot. It is a goal-oriented system where an LLM reasons, uses tools, maintains memory, creates plans, and takes actions.**

> **RAG gives Agent knowledge. Tools give Agent abilities. Planning gives Agent strategy. Memory gives Agent continuity.**

> **Modern enterprise AI applications are increasingly built by combining LLM reasoning, retrieval systems, and external actions into Agent architectures.**
