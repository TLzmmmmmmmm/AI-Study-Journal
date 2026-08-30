````markdown
# RAG Fundamentals Notes

> Week 2 · Day 1  
> Goal: Build a clear mental model of Retrieval-Augmented Generation before implementing a production RAG system.

---

# 1. Why RAG Exists

## 1.1 LLM Knowledge Boundary

A Large Language Model does **not automatically query a company database** when answering a question.

A basic LLM application looks like:

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

- model parameters learned during training
- system prompt
- conversation history
- current user message

It does **not automatically have access to private or updated company knowledge**.

For example:

```text
User:
What frequency range does Product X support?

LLM:
403–470 MHz
```

The answer may be:

1. correct because the model happened to learn it
2. inferred from a similar product
3. guessed
4. completely hallucinated

A fluent and specific answer is not necessarily a correct answer.

### Key Idea

```text
Plausible ≠ True
```

LLMs generate probable token sequences rather than performing exact database lookups.

---

## 1.2 Ungrounded vs Grounded Generation

### Ungrounded Generation

The model makes factual claims without explicit supporting evidence supplied to the current request.

```text
Question
   ↓
LLM internal knowledge
   ↓
Answer
```

Even if the answer is accidentally correct, it may still be ungrounded.

---

### Grounded Generation

The model generates an answer based on explicit trusted evidence.

```text
Company Document

"Product X supports 400–470 MHz."

        ↓

LLM Context

        ↓

Answer:
"Product X supports 400–470 MHz."
```

The answer can be traced back to a source.

### Important Distinction

```text
Correctness ≠ Groundedness
```

An answer can be:

| Correct | Grounded | Meaning |
|---|---|---|
| Yes | Yes | Ideal |
| Yes | No | Correct by model memory / luck |
| No | Yes | Source or reasoning may be wrong |
| No | No | Worst case |

---

## 1.3 Core Idea of RAG

RAG stands for:

```text
Retrieval-Augmented Generation
```

The idea is:

```text
User Question
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

RAG does **not normally train company knowledge into model parameters**.

Instead:

```text
External Knowledge
      ↓
Retrieval
      ↓
Current Context Window
      ↓
LLM Inference
```

### Core Definition

> RAG retrieves relevant external knowledge at inference time and injects that knowledge into the model's context before generation.

---

# 2. RAG Architecture

RAG should be understood as **two pipelines**.

---

## 2.1 Offline / Indexing Pipeline

```text
Company Knowledge
       ↓
Preprocessing
       ↓
Chunking
       ↓
Chunks
       ↓
Embedding
       ↓
Vectors
       ↓
Vector Index
```

This usually happens **before the user sends a query**.

Purpose:

> Prepare company knowledge so that it can be searched efficiently later.

---

## 2.2 Online / Query Pipeline

```text
User Query
       ↓
Query Embedding
       ↓
Query Vector
       ↓
Similarity Search
       ↓
Ranking
       ↓
Top-K Chunks
       ↓
Retrieve Original Text
       ↓
Context Injection
       ↓
LLM
       ↓
Grounded Answer
```

This happens while the user is waiting for a response.

---

## 2.3 Offline vs Online

### Offline

Usually includes:

- document ingestion
- preprocessing
- chunking
- document embedding
- vector index construction

The user does not wait for these operations during every request.

### Online

Usually includes:

- query embedding
- retrieval
- context construction
- LLM inference
- response streaming

These operations directly affect user-facing latency.

---

## 2.4 Indexing vs Retrieval

### Indexing

```text
Prepare knowledge
```

Example:

```text
Documents
→ Chunks
→ Embeddings
→ Index
```

### Retrieval

```text
Select relevant knowledge for the current query
```

Example:

```text
Query
→ Query Embedding
→ Similarity Search
→ Top-K
```

### Key Difference

```text
Indexing prepares knowledge.

Retrieval selects knowledge.
```

---

## 2.5 Retrieval vs Generation

### Retrieval

Responsible for:

```text
finding evidence
```

Output might simply be:

```text
Chunk A
Chunk B
Chunk C
```

### Generation

Responsible for:

```text
using evidence to formulate a natural-language answer
```

Example:

```text
Retrieved Chunks
      ↓
LLM
      ↓
"Based on the available company information..."
```

### Key Idea

```text
Retriever finds.

LLM writes.
```

---

# 3. Documents and Chunks

## 3.1 Why Documents Need Chunking

Suppose a product page contains:

```text
Overview
Features
Technical Parameters
Applications
Accessories
Support
```

If the entire page becomes one vector, retrieval granularity may be too coarse.

A user asking:

```text
What is the frequency range?
```

only needs a small part of the page.

Therefore:

```text
Document
   ↓
Chunks
```

---

## 3.2 What Is a Chunk?

A Chunk is usually the basic unit used for retrieval.

Example:

```text
Document: Product X

Chunk 1:
Product X Overview

Chunk 2:
Product X Features

Chunk 3:
Product X Frequency Specifications

Chunk 4:
Product X Battery Information
```

A good chunk should contain enough context to remain understandable.

---

## 3.3 Chunk Too Large

Possible problems:

```text
Too much unrelated information
↓
Semantic signal becomes less precise
↓
Retrieval precision may decrease
↓
More context tokens
```

---

## 3.4 Chunk Too Small

Possible problems:

```text
Missing context
Missing product identity
Incomplete facts
Broken semantic meaning
```

Example:

Bad chunk:

```text
Supports digital mode.
```

Question:

```text
Which product supports digital mode?
```

The chunk may no longer contain enough information to answer.

---

## 3.5 Chunking Is a Granularity Problem

The goal is not:

```text
make chunks as small as possible
```

or:

```text
make chunks as large as possible
```

The goal is:

> Create retrieval units that are semantically meaningful and sufficiently self-contained.

---

# 4. Original Text vs Embedding

Suppose a chunk contains:

```text
Product X supports 400–470 MHz.
```

After embedding:

```text
[0.13, -0.42, 0.78, ..., 0.05]
```

The vector helps the system determine:

```text
How semantically similar is this chunk to the query?
```

But the vector does **not replace the original factual text**.

---

## 4.1 Why Keep Original Text?

Because:

```text
Vector
→ used to FIND knowledge

Original Text
→ used to ANSWER the question
```

Full flow:

```text
Original Chunk
      │
      ├──→ Embedding → Vector
      │                  ↓
      │            Similarity Search
      │                  ↓
      └──────── Retrieve Original Chunk
                         ↓
                   LLM Context
                         ↓
                       Answer
```

### Key Idea

> Embeddings are semantic representations for retrieval, not replacements for the original knowledge.

---

# 5. Embeddings

## 5.1 What Is an Embedding?

An embedding maps text into a numerical vector.

```text
Text
 ↓
Embedding Model
 ↓
Vector
```

Example:

```text
"hotel wireless communication"

↓

[0.21, -0.42, 0.73, ..., 0.04]
```

The goal is to encode semantic relationships into a vector space.

---

## 5.2 Why Embeddings Help Retrieval

Consider:

```text
"酒店工作人员无线通信"
```

and:

```text
"宾馆员工数字对讲系统"
```

Keyword overlap is limited.

But their meanings are similar.

A good embedding model may produce vectors that are relatively close.

Therefore:

```text
Semantic similarity
        ↓
Vector similarity
        ↓
Semantic retrieval
```

---

# 6. Vector

A vector is an ordered list of numbers.

Example:

```text
[0.21, -0.42, 0.73]
```

Real embedding vectors may contain hundreds or thousands of dimensions.

Example:

```text
[x1, x2, x3, ..., xn]
```

---

## 6.1 Do Individual Dimensions Have Clear Meanings?

Usually no.

Do not assume:

```text
dimension 1 = hotel
dimension 2 = communication
dimension 3 = product
```

Embedding representations are generally **distributed representations**.

Meaning is encoded across many dimensions together.

---

# 7. Semantic Vector Space

Each embedding can be imagined as a point in a high-dimensional space.

Simplified 2D intuition:

```text
              Hotel Communication ●
                                  ● Hotel Radio

                   

                                      ● Customer Support

────────────────────────────────────────────→
```

Semantically similar texts tend to have more similar representations.

In reality, the space may contain hundreds or thousands of dimensions.

---

# 8. Token Embedding vs Retrieval Embedding

Both are called embeddings, but their roles differ.

---

## 8.1 Token Embedding

Used inside Transformer models.

```text
Token
 ↓
Embedding Representation
 ↓
Transformer Layers
```

Purpose:

> Represent individual tokens inside the model.

---

## 8.2 Retrieval / Text Embedding

Used in RAG.

```text
Sentence / Chunk
 ↓
Embedding Model
 ↓
Vector
```

Purpose:

- semantic search
- retrieval
- similarity comparison
- clustering

---

## 8.3 Main Difference

```text
Token Embedding
→ model-internal representation

Retrieval Embedding
→ search / retrieval representation
```

---

# 9. Document Embedding vs Query Embedding

During indexing:

```text
Document Chunk
      ↓
Embedding Model
      ↓
Document Vector
```

During user request:

```text
User Query
     ↓
Embedding Model
     ↓
Query Vector
```

Then:

```text
Query Vector
     ↓
Compare
     ↓
Document Vectors
```

---

## 9.1 Why Use the Same Compatible Embedding Model?

Documents and queries must exist in compatible vector spaces.

Bad example:

```text
Documents
→ Embedding Model A

Query
→ Unrelated Embedding Model B
```

Even if both return 768-dimensional vectors, the dimensions do not necessarily represent the same learned space.

Therefore:

```text
Documents + Queries
→ compatible embedding model
→ same semantic space
```

---

# 10. Similarity Search

After obtaining:

```text
Query Vector Q
```

and:

```text
Chunk Vector A
Chunk Vector B
Chunk Vector C
```

the system calculates:

```text
similarity(Q, A)
similarity(Q, B)
similarity(Q, C)
```

Then ranks them.

---

# 11. Cosine Similarity

A common similarity metric is:

```text
Cosine Similarity
```

Formula:

\[
\cos(\theta)
=
\frac{A \cdot B}
{\|A\|\|B\|}
\]

You do not need to memorize the derivation.

The important intuition is:

> Cosine similarity mainly measures how similar the directions of two vectors are.

---

## 11.1 Simple Example

```text
A = [1, 1]
B = [2, 2]
```

They have different lengths but the same direction.

Therefore their cosine similarity is very high.

---

## 11.2 Similarity Score Is Not Probability

If:

```text
similarity = 0.82
```

this does **not** mean:

```text
82% probability the answer is correct
```

Similarity is a ranking signal, not a calibrated probability.

Its meaning depends on:

- embedding model
- dataset
- query type
- similarity metric
- retrieval architecture

Therefore avoid blindly writing:

```python
if score < 0.7:
    reject()
```

Thresholds should be evaluated empirically.

---

# 12. Semantic Search

Semantic Search retrieves information based on meaning rather than exact keyword overlap.

Example:

```text
Query:
酒店员工无线通信

Document:
适用于宾馆工作人员使用的数字对讲系统
```

A keyword-only approach may struggle.

Semantic search may recognize that:

```text
酒店 ≈ 宾馆
员工 ≈ 工作人员
无线通信 ≈ 数字对讲
```

---

# 13. Semantic Search Limitations

Semantic Search is powerful, but it is not ideal for every query.

Possible weak cases:

```text
Product model numbers
Exact IDs
Technical codes
Exact numerical values
Abbreviations
Rare proper nouns
```

Example:

```text
P8668Ex
P8668
P8660
```

These identifiers may be semantically difficult to distinguish using dense embeddings alone.

Therefore production retrieval systems may later combine:

```text
Dense Retrieval
+
Keyword / Sparse Retrieval
+
Metadata Filtering
```

This leads to concepts such as Hybrid Search.

For Day 1, only understand that these methods exist.

---

# 14. Retrieval Pipeline

Basic dense retrieval:

```text
User Query
     ↓
Embedding
     ↓
Query Vector
     ↓
Similarity Search
     ↓
Ranking
     ↓
Top-K Chunks
```

The Retriever's job ends here.

---

# 15. Relevance

A chunk does not have a permanent relevance score.

Relevance depends on the query.

More accurately:

```text
Relevance(query, chunk)
```

Example:

For:

```text
"What product works well in hotels?"
```

a hotel solution chunk may rank highly.

For:

```text
"What is Product X's frequency?"
```

the same hotel solution chunk may become irrelevant.

---

# 16. Relevant Chunk vs Answer-Bearing Chunk

## Relevant Chunk

A chunk whose topic is related to the question.

Example:

```text
Product X is a professional digital radio.
```

Question:

```text
What frequency does Product X support?
```

The chunk is relevant.

But it does not contain the answer.

---

## Answer-Bearing Chunk

A chunk that contains evidence required to answer the question.

Example:

```text
Product X frequency range: 400–470 MHz.
```

This chunk is:

```text
Relevant
+
Answer-bearing
```

### Key Idea

Retrieval quality should eventually evaluate whether **answer-bearing chunks** appear in Top-K.

---

# 17. Top-K

Top-K means:

> Return the K highest-ranked chunks.

Example:

```text
Chunk A  0.91
Chunk B  0.84
Chunk C  0.76
Chunk D  0.45
Chunk E  0.20
```

If:

```text
K = 3
```

Retriever returns:

```text
A
B
C
```

---

## 17.1 K Too Small

Possible problem:

```text
Correct answer-bearing chunk
may not enter Top-K
```

This lowers retrieval recall.

---

## 17.2 K Too Large

Possible problems:

```text
Noise ↑
Tokens ↑
Cost ↑
Latency ↑
Conflicting information ↑
LLM confusion ↑
```

Therefore:

```text
Larger K ≠ Always Better
```

---

# 18. Top-1 Does Not Mean "Good"

Nearest-neighbor retrieval always ranks something first.

Suppose the database does not contain the requested information.

The system may still return:

```text
Rank 1
Rank 2
Rank 3
```

But:

```text
Best available result
≠
Good evidence
```

Analogy:

```text
Highest exam score = 32/100
```

Someone is still ranked first, but the score is still poor.

---

# 19. Unknown Queries

Suppose the user asks:

```text
Can Product X operate underwater at 50 meters continuously?
```

If the knowledge base contains no such information, the retriever may still return related Product X chunks.

However:

```text
related information
≠
sufficient evidence
```

Therefore the system needs a fallback mechanism.

---

# 20. Context Injection

After retrieval:

```text
User Question
+
Top-K Original Chunks
```

are placed into the LLM's current context.

Example:

```text
SYSTEM:
You are the company's AI support assistant.
Do not invent unsupported product facts.

CONTEXT:

[Source 1]
Product X ...

[Source 2]
Hotel solution ...

USER:
What product is suitable for hotel employees?
```

This is:

```text
Context Injection
```

---

# 21. System Prompt vs Retrieved Context

These have different roles.

## System Prompt

Defines:

```text
Role
Behavior
Rules
Policies
Constraints
```

Example:

```text
Do not invent unsupported specifications.
```

---

## RAG Context

Provides:

```text
Facts
Evidence
Product information
Company knowledge
```

Example:

```text
Product X supports 400–470 MHz.
```

### Key Distinction

```text
System Prompt
= How should the model behave?

RAG Context
= What evidence should the model use?
```

---

# 22. Minimal Sufficient Context

A good RAG system should aim for:

> The smallest amount of context that still contains everything necessary to answer correctly.

---

## 22.1 Too Little Context

Example:

```text
Power: 2W
```

Problem:

```text
Which product?
```

This may be:

```text
Minimal
but
Not Sufficient
```

---

## 22.2 Too Much Context

Example:

```text
Product X power
Product Y power
Product Z power
Company history
Support policy
Solution pages
...
```

The answer may be included, but:

```text
Noise ↑
Tokens ↑
Latency ↑
Confusion risk ↑
```

This is:

```text
Sufficient
but
Not Minimal
```

---

## 22.3 Ideal

```text
Product: X
Power: 2W
```

Contains enough identity and factual evidence.

### Key Idea

```text
Good RAG
≠
Retrieve everything

Good RAG
=
Retrieve minimal sufficient evidence
```

---

# 23. Why RAG Reduces Hallucination

Without RAG:

```text
Question
 ↓
Model Parameters
 ↓
Generation
```

The model may rely heavily on uncertain internal knowledge.

With RAG:

```text
Question
 ↓
Trusted External Evidence
 ↓
LLM Context
 ↓
Generation
```

This reduces reliance on uncertain model memory.

---

# 24. Why RAG Cannot Eliminate Hallucination

RAG itself introduces multiple possible failure points.

```text
Knowledge
 ↓
Ingestion
 ↓
Chunking
 ↓
Embedding
 ↓
Retrieval
 ↓
Context
 ↓
Generation
```

Any layer may fail.

---

# 25. RAG Failure Types

## 25.1 Knowledge Missing

The correct information does not exist in the knowledge source.

```text
No knowledge
→ Retriever cannot retrieve it
```

---

## 25.2 Knowledge Error

The company source itself contains incorrect or outdated information.

RAG may faithfully retrieve and reproduce incorrect data.

---

## 25.3 Ingestion Failure

The source contains correct information, but the ingestion pipeline does not include it.

```text
Website ✓
documents.jsonl ✗
```

---

## 25.4 Chunking Failure

The information exists in the normalized document but is broken or loses necessary context during chunking.

Example:

```text
Chunk 1:
Frequency: 400–

Chunk 2:
470 MHz
```

---

## 25.5 Retrieval Failure

The correct chunk exists in the index but does not enter Top-K.

```text
Correct Chunk exists ✓
Index contains it ✓
Top-K misses it ✗
```

---

## 25.6 Context Construction Failure

Retriever finds the correct chunk, but the backend does not correctly include it in the LLM request.

---

## 25.7 Generation Failure

The correct evidence exists in the LLM context, but the LLM still produces an incorrect answer.

Example:

```text
Context:
400–470 MHz

LLM:
403–470 MHz
```

---

# 26. Recommended RAG Debug Order

When an answer is wrong, do **not immediately modify the prompt**.

Debug in this order:

```text
1. Does the correct knowledge exist?
            ↓
2. Did ingestion capture it?
            ↓
3. Did chunking preserve it?
            ↓
4. Did the retriever find it?
            ↓
5. Did it enter the LLM context?
            ↓
6. Did the LLM use it correctly?
```

In short:

```text
Knowledge
→ Ingestion
→ Chunk
→ Retrieval
→ Context
→ Generation
```

### Key Engineering Principle

> An incorrect RAG answer is not automatically a prompt problem.

---

# 27. Fallback

If evidence is insufficient, the system should avoid guessing.

Example:

```text
The available company information does not contain enough evidence to confirm this specification.
```

Then optionally guide the user to:

```text
Support
Contact
Official Product Page
```

For enterprise customer support:

```text
Correct refusal
>
Confident hallucination
```

---

# 28. RAG vs System Prompt

## System Prompt

Best for:

```text
Behavior
Policy
Role
Style
Constraints
```

Example:

```text
Do not invent unsupported facts.
```

## RAG

Best for:

```text
Product specifications
Company information
Solutions
Support documentation
Frequently changing facts
```

### Summary

```text
System Prompt
= Rules

RAG
= Facts
```

---

# 29. RAG vs Long Context

Long Context answers:

> How much information can the model receive?

RAG answers:

> Which information should the model receive for this query?

Therefore:

```text
Long Context
= Capacity

RAG
= Selection
```

They can work together.

```text
Knowledge Base
 ↓
RAG
 ↓
Relevant Documents
 ↓
Long-Context LLM
```

---

# 30. Why Not Put All Company Knowledge in the Prompt?

Possible issues:

```text
Input tokens ↑
Cost ↑
Latency ↑
Context noise ↑
Conflicting facts ↑
Maintenance difficulty ↑
```

The goal is not:

```text
Give the model all available knowledge.
```

The goal is:

```text
Give the model the knowledge required for this question.
```

---

# 31. RAG vs Fine-Tuning

## RAG

```text
External Knowledge
      ↓
Retrieval
      ↓
Current Context
```

RAG changes:

```text
model input
```

---

## Fine-Tuning

```text
Training Examples
      ↓
Gradient Updates
      ↓
Model Parameters
```

Fine-tuning changes:

```text
model parameters
```

### Core Difference

```text
RAG
= changes input

Fine-tuning
= changes parameters
```

---

# 32. When RAG Is Usually Better

RAG is especially useful for:

- changing product information
- internal company documents
- product specifications
- company policies
- technical support documentation
- source citation
- private knowledge
- frequently updated knowledge

---

# 33. When Fine-Tuning Is Usually Better

Fine-tuning may be more useful for:

- consistent behavior patterns
- specialized task execution
- output formatting
- classification behavior
- domain-specific writing style
- repeated structured tasks

---

# 34. RAG and Fine-Tuning Can Work Together

Example:

```text
Fine-Tuned Model
→ customer-service behavior

+

RAG
→ current company knowledge
```

Conceptually:

```text
Fine-tuning
→ How the model tends to behave

RAG
→ What facts the model should use now
```

---

# 35. Architecture Comparison

| Method | Main Purpose |
|---|---|
| System Prompt | Rules / behavior |
| Long Context | Input capacity |
| RAG | Knowledge selection and grounding |
| Fine-tuning | Parameter / behavior adaptation |

---

# 36. Separation of Concerns

A maintainable RAG architecture separates responsibilities.

```text
System Prompt
→ Policy

Knowledge Base
→ Facts

Retriever
→ Evidence Selection

LLM
→ Language Generation
```

Therefore:

```text
Wrong policy
→ modify prompt

Wrong fact
→ modify knowledge

Wrong evidence
→ debug retriever

Wrong final wording/fact use
→ debug generation
```

---

# 37. Vector Database

A Vector Database / Vector Index is responsible for:

```text
Store vectors
+
Search nearest vectors efficiently
```

It does **not** create embeddings.

Compare:

```text
Embedding Model
→ Text → Vector

Vector Database
→ Store / Search Vectors
```

### Important

```text
Embedding Model ≠ Vector Database
```

---

# 38. Basic RAG Data Record

Conceptually, a stored chunk may contain:

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
→ LLM evidence

metadata
→ filtering / debugging / citation
```

---

# 39. Evaluation Foundations

Before building RAG, establish a baseline.

```text
Frozen Questions
      ↓
V0 Direct LLM
      ↓
Results
```

After building RAG:

```text
Same Frozen Questions
      ↓
V1 RAG
      ↓
Results
```

Then compare:

```text
V0 vs V1
```

---

# 40. Ground Truth

Ground Truth should come from trusted company sources.

Not from the model being tested.

Correct:

```text
Company Source
     ↓
Expected Answer
```

Incorrect:

```text
LLM Answer
     ↓
Expected Answer
```

---

# 41. Basic V0 Metrics

For the first baseline, useful metrics include:

## Correctness

```text
Is the answer factually correct?
```

## Hallucination

```text
Did the model introduce unsupported company facts?
```

## Appropriate Refusal

For an unanswerable question:

```text
Did the model correctly refuse instead of guessing?
```

---

# 42. Retrieval Metrics Preview

These will be studied more deeply later.

## Hit@K

Did at least one correct answer-bearing chunk appear in Top-K?

Example:

```text
Expected Chunk = G

Top-3:
A
G
C
```

Then:

```text
Hit@3 = Yes
```

---

## Recall@K

If multiple chunks are required:

```text
Expected:
A B C

Retrieved:
A C D E
```

Then:

```text
Recall = 2 / 3
```

For Day 1, only understand the intuition.

---

# 43. Complete RAG Mental Model

```text
                     OFFLINE

Company Knowledge
       ↓
Preprocessing
       ↓
Chunking
       ↓
Chunks
       ↓
Embedding Model
       ↓
Chunk Vectors
       ↓
Vector Index
       │
       │
       └────────────────────┐
                            │
                            ↓
                         ONLINE

User Query
       ↓
Query Embedding
       ↓
Query Vector
       ↓
Similarity Search ← Vector Index
       ↓
Ranking
       ↓
Top-K
       ↓
Answer-Bearing Chunks
       ↓
Retrieve Original Text
       ↓
Minimal Sufficient Context
       ↓
LLM
       ↓
Grounded Answer
       ↓
Sources
```

---

# 44. Eight Core Statements

If most details are forgotten later, remember these:

1. **LLMs do not automatically access company knowledge.**

2. **RAG retrieves external knowledge at inference time.**

3. **Indexing prepares knowledge; retrieval selects knowledge.**

4. **Embeddings represent semantic information as vectors.**

5. **Similarity search ranks chunks relative to the current query.**

6. **Vectors are used to find knowledge; original text provides factual evidence.**

7. **Retrieval finds evidence; the LLM generates the answer.**

8. **Good RAG aims for minimal sufficient context.**

---

# 45. Common Misconceptions

## Misconception 1

```text
RAG teaches new knowledge to the model.
```

Wrong.

RAG normally provides external knowledge through context during inference.

---

## Misconception 2

```text
Higher similarity means higher probability of correctness.
```

Wrong.

Similarity is a ranking signal, not calibrated correctness probability.

---

## Misconception 3

```text
Top-1 means the result is good.
```

Wrong.

Top-1 only means it is the best result among available candidates.

---

## Misconception 4

```text
Higher K is always better.
```

Wrong.

Higher K may improve recall but increase noise, cost, and confusion.

---

## Misconception 5

```text
Embedding replaces original text.
```

Wrong.

Embedding helps find the text. The original text still provides the evidence.

---

## Misconception 6

```text
RAG completely eliminates hallucination.
```

Wrong.

RAG introduces several possible failure stages and cannot guarantee correctness.

---

## Misconception 7

```text
A wrong RAG answer means the prompt is bad.
```

Wrong.

Always debug:

```text
Knowledge
→ Ingestion
→ Chunking
→ Retrieval
→ Context
→ Generation
```

---

# 46. Day 1 Final Summary

A basic LLM application is:

```text
User
 ↓
LLM
 ↓
Answer
```

A basic RAG application becomes:

```text
User
 ↓
Retriever
 ↓
External Knowledge
 ↓
Relevant Chunks
 ↓
Context
 ↓
LLM
 ↓
Grounded Answer
```

The central engineering goal is not merely:

```text
make the LLM smarter
```

but:

```text
provide the right evidence
at the right time
in the right amount
```

A strong RAG system therefore depends on the entire pipeline:

```text
Knowledge Quality
+
Document Representation
+
Chunking
+
Embeddings
+
Retrieval
+
Context Construction
+
Generation
+
Evaluation
```

The most important Day 1 lesson:

> **RAG is not just "LLM + Vector Database." It is a knowledge retrieval and grounding architecture whose quality depends on every stage between raw company knowledge and the final answer.**
````
