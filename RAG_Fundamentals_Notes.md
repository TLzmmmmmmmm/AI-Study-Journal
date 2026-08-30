````markdown
# RAG Fundamentals Notes

> Week 2 · Day 1  
> Goal: Understand the core mental model of Retrieval-Augmented Generation before implementation.

---

# 1. Why RAG Exists

LLMs do not automatically query company databases.

A basic LLM application:

```text
User Question
     ↓
System Prompt
     ↓
LLM
     ↓
Answer
```

The model mainly relies on:

- model parameters
- system prompt
- conversation history
- current user message

A fluent answer may still be incorrect.

```text
Plausible ≠ True
```

---

## 1.1 Grounded vs Ungrounded

### Ungrounded

The model makes factual claims without explicit supporting evidence.

```text
Question
↓
LLM Memory
↓
Answer
```

### Grounded

The answer is supported by trusted evidence provided in the current context.

```text
Company Document
↓
Retrieved Evidence
↓
LLM
↓
Answer
```

Important:

```text
Correctness ≠ Groundedness
```

An answer can be correct by luck or model memory but still be ungrounded.

---

## 1.2 Core Idea of RAG

RAG = Retrieval-Augmented Generation.

```text
Question
   ↓
Retrieve External Knowledge
   ↓
Relevant Evidence
   ↓
Add Evidence to Context
   ↓
LLM
   ↓
Grounded Answer
```

RAG usually does **not** train company knowledge into model parameters.

Instead:

```text
External Knowledge
↓
Retrieval
↓
Current Context
↓
LLM Inference
```

> RAG retrieves relevant external knowledge at inference time and injects it into the model's context.

---

# 2. RAG Architecture

RAG consists of two main pipelines.

## 2.1 Offline: Indexing

```text
Documents
   ↓
Preprocessing
   ↓
Chunking
   ↓
Embeddings
   ↓
Vector Index
```

Purpose:

> Prepare knowledge before users ask questions.

---

## 2.2 Online: Query Pipeline

```text
User Query
   ↓
Query Embedding
   ↓
Similarity Search
   ↓
Top-K Chunks
   ↓
Original Text
   ↓
Context Injection
   ↓
LLM
   ↓
Grounded Answer
```

### Key Differences

```text
Indexing
= prepare knowledge

Retrieval
= select knowledge

Generation
= use knowledge to write the answer
```

---

# 3. Documents and Chunks

A long document is usually divided into smaller retrieval units called **chunks**.

```text
Document
├── Overview
├── Features
├── Technical Parameters
└── Applications
```

Why chunk?

Because users usually need only one part of a document.

### Chunk Too Large

```text
More noise
Less precise retrieval
More context tokens
```

### Chunk Too Small

```text
Missing context
Missing product identity
Incomplete information
```

Goal:

> Create chunks that are semantically meaningful and sufficiently self-contained.

---

# 4. Original Text vs Embedding

Example chunk:

```text
Product X supports 400–470 MHz.
```

Embedding:

```text
[0.13, -0.42, 0.78, ...]
```

Their roles are different:

```text
Vector
→ used to find knowledge

Original Text
→ used as evidence for the LLM
```

Therefore the original text must still be stored after embedding.

---

# 5. Embeddings

An embedding maps text into a numerical vector.

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

Semantically similar texts tend to have similar vector representations.

Example:

```text
酒店工作人员无线通信

宾馆员工数字对讲系统
```

The keywords differ, but the meanings are similar.

This enables:

```text
Semantic Similarity
↓
Semantic Search
```

---

## 5.1 Token Embedding vs Retrieval Embedding

### Token Embedding

Used inside Transformer models.

```text
Token
↓
Internal Representation
```

### Retrieval Embedding

Used for search.

```text
Sentence / Chunk
↓
Embedding Model
↓
Vector
```

Main difference:

```text
Token Embedding
→ model internal representation

Retrieval Embedding
→ search representation
```

---

# 6. Document vs Query Embedding

During indexing:

```text
Document Chunk
↓
Embedding Model
↓
Document Vector
```

During a user request:

```text
Query
↓
Embedding Model
↓
Query Vector
```

Then compare:

```text
Query Vector
vs
Document Vectors
```

Documents and queries normally need compatible embedding models so they exist in the same semantic vector space.

---

# 7. Similarity Search

A common metric is **Cosine Similarity**.

```text
similarity(Query Vector, Chunk Vector)
```

The basic intuition:

> Cosine similarity compares the direction of vectors.

Higher similarity usually means stronger semantic similarity.

However:

```text
similarity = 0.82
```

does **not** mean:

```text
82% probability of correctness
```

Similarity is a ranking signal, not a calibrated probability.

---

# 8. Semantic Search

Semantic Search retrieves based on meaning rather than only exact keywords.

Example:

```text
Query:
酒店员工无线通信

Document:
适用于宾馆工作人员的数字对讲系统
```

A semantic retriever may still match them.

However, dense semantic retrieval can be weaker for:

- exact product IDs
- model numbers
- numerical values
- rare technical terms

Later systems may combine semantic search with keyword search or metadata filtering.

---

# 9. Retrieval

Basic retrieval:

```text
Query
↓
Embedding
↓
Similarity Search
↓
Ranking
↓
Top-K
```

Retriever output:

```text
Chunk A
Chunk B
Chunk C
```

The Retriever does **not** write the final answer.

```text
Retrieval
= find evidence

Generation
= write answer
```

---

# 10. Relevant vs Answer-Bearing Chunk

### Relevant Chunk

Related to the topic.

Example:

```text
Product X is a digital radio.
```

Question:

```text
What is Product X's frequency?
```

Relevant, but does not contain the answer.

### Answer-Bearing Chunk

Contains the actual evidence needed.

```text
Product X frequency: 400–470 MHz.
```

A good retriever should place answer-bearing chunks in Top-K.

---

# 11. Top-K

Top-K means returning the K highest-ranked chunks.

Example:

```text
1. Chunk A
2. Chunk B
3. Chunk C
```

If:

```text
K = 3
```

the retriever returns all three.

### K Too Small

```text
May miss correct evidence
```

### K Too Large

```text
Noise ↑
Tokens ↑
Latency ↑
Conflicting information ↑
```

Therefore:

```text
Larger K ≠ Always Better
```

---

# 12. Top-1 Does Not Mean "Good"

Top-1 only means:

> Best result among available candidates.

It does not guarantee:

```text
The result is relevant
or
The result contains enough evidence
```

Even when the knowledge base has no answer, the retriever can still rank something first.

---

# 13. Minimal Sufficient Context

Good RAG aims to provide:

> The smallest amount of context that still contains all evidence needed to answer correctly.

### Too Little

```text
Power: 2W
```

Missing:

```text
Which product?
```

### Too Much

```text
Product X
Product Y
Product Z
Company history
Support information
...
```

Problems:

```text
Noise
Tokens
Latency
Confusion
```

Ideal:

```text
Product: X
Power: 2W
```

Core principle:

```text
Good RAG
≠ retrieve everything

Good RAG
= retrieve minimal sufficient evidence
```

---

# 14. System Prompt vs RAG Context

## System Prompt

Defines:

```text
Role
Behavior
Rules
Policies
```

Example:

```text
Do not invent unsupported facts.
```

## RAG Context

Provides:

```text
Facts
Evidence
Product Information
Company Knowledge
```

Key distinction:

```text
System Prompt
= How should the model behave?

RAG Context
= What evidence should the model use?
```

---

# 15. Why RAG Reduces Hallucination

Without RAG:

```text
Question
↓
Model Memory
↓
Answer
```

With RAG:

```text
Question
↓
Trusted Evidence
↓
Context
↓
Answer
```

RAG reduces reliance on uncertain model memory.

However, RAG cannot guarantee correctness.

---

# 16. RAG Failure Types

## Knowledge Missing

Correct information does not exist in the source.

## Ingestion Failure

The source contains the information, but it was not imported.

## Chunking Failure

The information exists but is broken or loses context.

## Retrieval Failure

The correct chunk exists but does not enter Top-K.

## Context Construction Failure

The retriever finds the correct chunk, but it is not correctly sent to the LLM.

## Generation Failure

The LLM sees the correct evidence but still answers incorrectly.

---

## Recommended Debug Order

```text
Knowledge
   ↓
Ingestion
   ↓
Chunking
   ↓
Retrieval
   ↓
Context
   ↓
Generation
```

> A wrong RAG answer is not automatically a prompt problem.

---

# 17. Fallback

If retrieved evidence is insufficient:

```text
Do not guess.
```

Instead:

```text
The available company information does not contain enough evidence to confirm this.
```

For enterprise customer support:

```text
Correct Refusal
>
Confident Hallucination
```

---

# 18. RAG vs Alternatives

## System Prompt

```text
Main purpose:
Rules / Behavior
```

## Long Context

```text
Main purpose:
Input Capacity
```

## RAG

```text
Main purpose:
Knowledge Selection + Grounding
```

## Fine-Tuning

```text
Main purpose:
Parameter / Behavior Adaptation
```

Key summary:

```text
System Prompt = rules

Long Context = capacity

RAG = knowledge selection

Fine-tuning = parameter adaptation
```

---

# 19. RAG vs Fine-Tuning

RAG:

```text
External Knowledge
↓
Current Context
```

Fine-tuning:

```text
Training Data
↓
Gradient Updates
↓
Model Parameters
```

Therefore:

```text
RAG
= changes input

Fine-tuning
= changes parameters
```

Frequently changing company facts are generally better suited to RAG.

---

# 20. Vector Database

A Vector Database / Vector Index:

```text
stores vectors
+
searches nearby vectors efficiently
```

It does not create embeddings.

```text
Embedding Model
→ Text → Vector

Vector Database
→ Store / Search Vectors
```

---

# 21. Basic RAG Data Record

A chunk may conceptually contain:

```json
{
  "chunk_id": "product:x:frequency",
  "text": "Product X supports 400–470 MHz.",
  "vector": [0.12, -0.41, 0.78],
  "metadata": {
    "product": "Product X",
    "section": "frequency",
    "source_url": "/products/x"
  }
}
```

Roles:

```text
vector
→ retrieval

text
→ evidence

metadata
→ filtering / citation / debugging
```

---

# 22. Evaluation Foundations

Before RAG:

```text
Frozen Questions
↓
V0 Direct LLM
↓
Baseline Results
```

After RAG:

```text
Same Questions
↓
V1 RAG
↓
Results
```

Then compare:

```text
V0 vs V1
```

Ground truth should come from trusted company sources, not from the LLM being evaluated.

Basic metrics:

- Correctness
- Hallucination
- Appropriate Refusal

Later retrieval metrics:

- Hit@K
- Recall@K

---

# 23. Complete Mental Model

```text
                    OFFLINE

Company Knowledge
       ↓
Preprocessing
       ↓
Chunking
       ↓
Embeddings
       ↓
Vector Index
       │
       │
       └───────────────┐
                       ↓
                    ONLINE

User Query
       ↓
Query Embedding
       ↓
Similarity Search
       ↓
Top-K
       ↓
Original Text
       ↓
Minimal Sufficient Context
       ↓
LLM
       ↓
Grounded Answer
```

---

# 24. Eight Core Statements

1. LLMs do not automatically access company knowledge.
2. RAG retrieves external knowledge at inference time.
3. Indexing prepares knowledge; retrieval selects knowledge.
4. Embeddings represent semantic information as vectors.
5. Similarity search ranks chunks relative to the query.
6. Vectors find knowledge; original text provides factual evidence.
7. Retrieval finds evidence; the LLM generates the answer.
8. Good RAG aims for minimal sufficient context.

---

# 25. Day 1 Final Summary

A basic LLM system:

```text
User
↓
LLM
↓
Answer
```

A basic RAG system:

```text
User
↓
Retriever
↓
Relevant Knowledge
↓
Context
↓
LLM
↓
Grounded Answer
```

The goal of RAG is not simply:

```text
give the LLM more information
```

but:

```text
give the right evidence
at the right time
in the right amount
```

The most important debugging principle:

```text
Knowledge
→ Ingestion
→ Chunking
→ Retrieval
→ Context
→ Generation
```

> RAG is not simply "LLM + Vector Database." It is a knowledge retrieval and grounding architecture.
````
