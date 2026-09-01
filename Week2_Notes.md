# Day 2 — Knowledge Ingestion: One-Sentence Notes

## Step 1 — Knowledge Sources & Document Boundaries

> **A Normalized Document represents a stable knowledge entity, while website source files are only the formats in which that knowledge happens to be stored.**

Normalized Document 应该围绕独立、可维护的 Knowledge Entity 建立，而不是按照 JSON、Markdown、Astro 等原始文件格式决定边界。

---

## Step 2 — Knowledge Inclusion & Exclusion

> **Facts belong in text, operational identifiers belong in metadata, and presentation noise or unsupported inference should be excluded.**

决定信息是否进入 Knowledge Base 时，应区分真实知识、系统 metadata、UI/SEO 噪声以及未经 Source 支持的推断。

---

## Step 3 — Knowledge Document Schema

> **A Knowledge Document combines identity, knowledge text, metadata, provenance, and change tracking into one stable RAG contract.**

Normalized Document 不只是正文，而是一个统一的数据契约：

**Knowledge Document = Identity + Text + Metadata + Provenance + Change Tracking**

---

## Step 4 — Text vs Metadata

> **Text contains what the model should understand semantically; metadata contains what the system should know and operate on exactly.**

`text` 主要服务于 Embedding 和 LLM，而 `metadata` 主要服务于 filtering、grouping、debugging、citation 和 evaluation。

Retrieval-critical identity 可以同时存在于两者中。

---

## Step 5 — Normalization

> **Normalization changes representation, not knowledge.**

Normalization 可以重组、标准化和连接明确的可信 Source，但不能总结、改写、推断或创造新的知识。

---

## Step 6 — Knowledge Ingestion Pipeline

> **Validate early, transform deterministically, keep pipeline stages separate, and write only validated knowledge artifacts.**

Day 2 的核心 pipeline 是：

**Load → Validate Sources → Validate Relationships → Normalize → Validate Documents → Serialize → documents.jsonl**

---

## Step 7 — Validation & Acceptance

> **Only validated, deterministic, traceable, and factually inspected knowledge should enter the retrieval pipeline.**

Schema 正确并不代表事实一定正确，因此最终 Knowledge Base 需要同时通过机器 validation 和人工 representative inspection。

---

# Day 2 Core Mental Model

> **Knowledge ingestion converts heterogeneous website content into a unified, deterministic, and trustworthy knowledge layer for downstream RAG.**

Knowledge Ingestion 的本质不是“把网站内容复制出来”，而是：

**把可信 Website Knowledge 转换成稳定、统一、可验证的 RAG Knowledge Documents。**

核心流程：

**Website Source → Curated Knowledge Source → Normalization → Validated Documents → documents.jsonl**

最终原则：

**Trusted Facts → Structured Knowledge → Validated Artifact**


# Day 3 — Chunking Strategy: One-Sentence Notes

## Step 1 — Document vs Chunk

> **Knowledge being present in a document does not guarantee that it is easy to retrieve.**

知识存在于 Document 中，并不意味着它天然容易被检索；Chunk 的作用是把知识转换成合适的 retrieval units。

---

## Step 2 — Chunking Trade-off

> **A chunk should be as small as possible to stay semantically focused, but as large as necessary to remain self-contained and answer-bearing.**

Chunk 应该尽可能小以保持语义聚焦，但又必须足够大，以保证上下文完整并能够承载答案。

---

## Step 3 — Three Chunking Strategies

> **Prefer semantic boundaries; use size limits as guardrails rather than the primary definition of knowledge units.**

优先按照语义边界切分知识，长度限制应该只是安全边界，而不应该决定知识单元本身。

---

## Step 4 — Chunk Boundary Design

> **A good chunk boundary follows the smallest trustworthy semantic structure that remains independently meaningful.**

好的 Chunk Boundary 应该遵循最小的、可信的、同时仍能独立表达完整语义的知识结构。

---

## Step 5 — Identity Preservation & Overlap

> **Use no mechanical overlap across natural semantic boundaries; repeat only necessary parent identity, and introduce limited overlap only when an oversized section must be split across an unavoidable non-semantic boundary.**

自然语义边界之间不机械添加 overlap；只重复必要的 Parent Identity，并仅在无法避免的非语义切分中使用有限 overlap。

---

## Step 6 — Chunk Schema & IDs

> **Chunk IDs should represent stable semantic identity, not random identity or mutable content.**

Chunk ID 应该表达稳定的语义身份，而不是随机 UUID，也不应该依赖可能变化的具体内容。

---

## Step 7 — Chunking Acceptance

> **A chunking strategy is ready when it consistently turns trusted documents into deterministic, self-contained, low-noise retrieval units without losing or inventing knowledge.**

当 Chunking Strategy 能稳定地把可信 Document 转换成确定性、自包含、低噪声的 Retrieval Units，同时既不丢失知识也不创造知识时，它才真正可以进入下一阶段。

---

# Day 3 Core Mental Model

> **Chunking is not about cutting text into smaller pieces; it is about designing the units that retrieval operates on.**

Chunking 的本质不是“把文字切短”，而是：

**定义 Retrieval 应该以什么知识粒度工作。**

核心目标：

**Good Chunk = Correct Evidence + Enough Context + Semantic Focus − Irrelevant Noise**

最终原则：

**Document = Knowledge Entity**

**Chunk = Retrieval Unit**


# Day 4 — Embedding + Semantic Retrieval: One-Sentence Notes

## Step 1 — Embedding Mental Model

> **Embedding maps text into a shared vector space where semantic relationships can be compared numerically.**

Embedding 的本质是把文本转换成用于语义比较的 Vector；Vector 用来寻找 Evidence，而 Original Chunk Text 才是真正的事实证据。

---

## Step 2 — Offline Chunk Embedding

> **Knowledge chunks are embedded during indexing and reused until either their content or the embedding-space configuration changes.**

Chunk Embedding 通常在 Offline Indexing 阶段提前生成，并通过 `chunk_id + content_hash + embedding configuration` 判断已有 Vector 是否可以复用。

核心变化处理：

**Unchanged → Reuse · Changed → Re-embed · New → Embed · Deleted → Remove**

---

## Step 3 — Query Embedding & Shared Embedding Space

> **Semantic retrieval works because queries and knowledge chunks are encoded into a compatible retrieval space where geometric similarity represents how well a chunk matches the query’s information need.**

Query 和 Chunk 能够比较，不是因为文字必须相同，而是因为它们被映射到 Compatible Embedding Space 中，使用户的信息需求可以和 Knowledge Evidence 进行几何比较。

核心原则：

**Same Dimensions ≠ Compatible Embedding Space**

---

## Step 4 — Cosine Similarity & Top-K

> **Semantic search ranks chunks by vector similarity and returns the Top-K nearest candidates, but nearest does not necessarily mean relevant, answer-bearing, or correct.**

Cosine Similarity 主要比较 Vector Direction，并据此对所有 Chunk 排名；Top-K 决定最终有多少 Retrieval Candidates 可以进入下一阶段。

需要牢记：

**Similarity Score ≠ Probability**

**Nearest ≠ Relevant**

**Relevant ≠ Answer-bearing**

---

## Step 5 — Retrieval Evaluation & Debugging

> **Retrieval quality should be measured by whether the correct answer-bearing evidence consistently appears near the top of the ranking, not by how high the similarity scores look.**

Retriever 的质量应该通过 Ground Truth 与实际 Top-K 结果比较，而不是观察 Similarity Score 是否“看起来很高”。

核心指标：

- **Hit@K** — Top-K 中是否至少找到一个正确 Evidence
- **Recall@K** — 所需 Evidence 被找回了多少
- **First Relevant Rank** — 第一个 Answer-bearing Evidence 排在第几位

核心目标：

**High Answer-Bearing Evidence Recall at a Small, Low-Noise K**

---

## Step 6 — Exact Entity Handling

> **Exact entity handling supplies high-precision identity signals, while dense retrieval supplies semantic intent signals; combining them improves model-specific queries without sacrificing broader retrieval recall.**

Exact Entity Handling 负责高精度判断“用户明确在说谁”，Dense Retrieval 负责理解“用户想知道什么”。

二者的分工可以简化为：

**Exact Entity → Who**

**Dense Retrieval → What**

因此 V1 使用：

**Entity-Prioritized Candidates + Dense Ranking / Fill**

而不是严格 Entity Filter。

---

## Step 7 — Vector Index & Retrieval Acceptance

> **A vector index optimizes how efficiently nearest neighbors are found at scale; it does not improve the knowledge, chunking, embeddings, or semantic quality of retrieval itself.**

NumPy Exact Search、FAISS 和 Vector Store 的主要区别在于 Vector 的存储、索引与搜索效率，而不是改变 Semantic Retrieval 的基本原理。

当前 V1：

**NumPy Exact Search → simple, transparent, exact**

未来规模增长后：

**FAISS / Vector Index → optimize nearest-neighbor search**

Vector Index 优化的是 Search Execution，而不是 Retrieval Semantics。

---

# Day 4 Core Mental Model

> **Semantic retrieval converts queries and knowledge chunks into a compatible vector space, ranks chunks by semantic proximity plus reliable exact signals, and returns a small set of answer-bearing evidence candidates.**

Semantic Retrieval 的核心流程：

**Chunk → Embedding → Chunk Vector**

**Query → Embedding → Query Vector**

然后：

**Query Vector + Chunk Vectors → Similarity → Ranking → Top-K**

Exact Entity Handling 提供额外的 Identity Signal：

**Exact Signal → Who**

**Semantic Signal → What**

最终原则：

**Embedding determines how knowledge is represented.**

**Similarity determines how vectors are compared.**

**Exact Entity Handling protects identity precision.**

**Top-K determines which evidence survives retrieval.**

**Vector Index determines how efficiently retrieval executes at scale.**

最终输出：

**Query → Top-K Answer-Bearing Evidence Candidates**

Day 4 到这里停止：

**Retrieval ≠ Generation**

