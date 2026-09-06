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

# Day 5 — RAG Generation Integration: One-Sentence Notes

## Step 1 — Retrieval vs Conversation History

> **Retrieval should search for evidence using the user’s current information need, while conversation history should help the generator interpret that evidence in context.**

V1 中 Retriever 只使用最后一条 User Message 进行检索，而有限 Conversation History 交给 LLM 理解多轮语境，避免历史内容污染当前 Retrieval Query。

核心分工：

**Latest User Message → Retrieval**

**Bounded Conversation History → Generation Context**

---

## Step 2 — RAG Layering

> **Keep retrieval, context construction, prompt construction, and generation as separate layers so each stage has one clear responsibility.**

V1 使用轻量分层：

**Retriever → Context Builder → Prompt Builder → LLM → Stream**

Retriever 负责找 Evidence，Context Builder 负责组织 Evidence，Prompt Builder 负责定义 LLM 应该如何使用 Evidence，而路由层只负责 orchestration。

---

## Step 3 — Retrieved Context as Evidence

> **Retrieved knowledge is untrusted evidence for answering the user, not instructions that can redefine system behavior.**

Retrieved Chunk 应该作为独立的 `CONTEXT` 输入，而不是直接拼入 System Prompt，更不能让 Knowledge Base 中的文字覆盖系统身份、安全规则或回答边界。

核心 Trust Boundary：

**System Instructions > Retrieved Evidence > User Request**

---

## Step 4 — Grounded Generation

> **The generator should answer from retrieved evidence when evidence exists, and explicitly abstain when the available evidence is insufficient.**

RAG 的目标不是让 LLM “知道更多”，而是限制 LLM：

**Evidence Available → Grounded Answer**

**Evidence Insufficient → Explicit Abstention**

不能用模型自身知识补全公司事实、产品参数、库存、价格、资质或其他未提供的信息。

---

## Step 5 — Prompt Responsibility

> **The prompt should define stable reasoning and grounding rules, while factual company knowledge should remain in the retrieved context rather than being duplicated into system instructions.**

System Prompt 负责：

**Identity + Trust Rules + Grounding + Safety + Fallback**

Retrieved Context 负责：

**Company Facts + Product Facts + Solution Facts + Support Facts**

Prompt 不应该变成第二个 Knowledge Base。

---

## Step 6 — Streaming RAG Integration

> **RAG changes how evidence is prepared before generation, but it does not require changing the existing HTTP streaming contract.**

V1 保留已有：

**POST `/api/chat-stream`**

和：

**NDJSON: `delta → ... → done/error`**

RAG 只插入 Generation 前：

**User Query → Retrieval → Context → Prompt**

而不改变前端已经依赖的 Streaming Protocol。

---

## Step 7 — Production Boundaries

> **A minimal production RAG system should add only the components required to improve grounding, while postponing complexity that has not yet been justified by evidence.**

V1 明确不加入：

* Query Rewriting
* Reranker
* Hybrid Search
* BM25
* Vector Database
* Multi-Agent Workflow
* LangChain / LangGraph

先验证：

**Simple Retrieval + Good Evidence + Strong Grounding**

是否已经足够。

---

# Day 5 Core Mental Model

> **RAG generation is the controlled handoff from retrieved evidence to the LLM, with strict separation between instructions, evidence, conversation context, and generation.**

Day 5 的核心链路：

**Latest User Message**

↓

**Retriever**

↓

**Top-K Evidence**

↓

**Context Builder**

↓

**Prompt Builder + Conversation History**

↓

**LLM**

↓

**NDJSON Streaming**

核心分工：

**Retriever decides what evidence enters.**

**Context Builder decides how evidence is represented.**

**Prompt decides how evidence may be used.**

**LLM converts evidence into a user-facing answer.**

最终原则：

**Retrieve Facts → Constrain Generation → Stream Grounded Answer**

---

# Day 6 — RAG Evaluation: One-Sentence Notes

## Step 1 — Evaluation Contract

> **A RAG system should be evaluated against explicit ground truth and acceptance criteria defined before inspecting its outputs.**

Evaluation 不能先看结果再决定什么叫“正确”，而应该提前定义：

**Query + Expected Evidence + Expected Behavior + Acceptance Rule**

从而避免人为根据模型输出修改评分标准。

---

## Step 2 — Evaluation Dataset Design

> **A useful evaluation dataset should represent real information needs, difficult boundaries, unknown questions, and failure modes rather than merely generating easy questions from existing chunks.**

Evaluation Query 不应该由 Chunk 机械反推，否则会天然偏向 Retriever。

数据集应该覆盖：

**Entity Questions + Parameter Questions + Recommendations + Solutions + Support + Company Facts + Unknowns**

并主动加入歧义、边界和容易混淆的 Case。

---

## Step 3 — Ground Truth Completeness

> **Ground truth should identify every chunk that can independently or jointly provide the required evidence, not just one convenient expected chunk.**

如果多个 Chunk 都能完整支持答案，它们都应该进入 Ground Truth。

否则：

```text
Retriever 找到了正确 Evidence
≠
Evaluation 一定判正确
```

Ground Truth 的质量决定 Metric 是否可信。

---

## Step 4 — Retrieval Metrics

> **Retrieval evaluation measures whether answer-bearing evidence appears early and reliably, not whether similarity scores are numerically high.**

核心指标：

* **Hit@K** — Top-K 是否至少出现正确 Evidence
* **Recall@K** — 所有标注 Evidence 被找回多少
* **First Relevant Rank** — 第一个 Relevant Chunk 在哪里
* **MRR** — Relevant Evidence 是否持续靠前

即使：

**Hit@5 = 100%**

如果正确 Evidence 总在 Rank 4–5，Retriever 仍然可能排序较弱。

---

## Step 5 — Generation Evaluation

> **Generation quality should be evaluated separately from retrieval quality because correct evidence can still produce a wrong answer, and weak retrieval can sometimes accidentally produce a plausible answer.**

Generation 应至少检查：

**Correctness**

**Groundedness**

**Completeness**

**Refusal Behavior**

**Hallucination**

因此：

**Retrieval Success ≠ Generation Success**

必须分别测量。

---

## Step 6 — Unknown & Abstention Evaluation

> **Unknown questions are first-class evaluation cases because a trustworthy RAG system must know when its evidence is insufficient.**

对于 Knowledge Base 不支持的问题：

**Correct Behavior = Refuse / Cannot Confirm**

而不是：

**Retrieve Similar Chunk → Guess an Answer**

Unknown Cases 是检验 Hallucination Control 的重要部分。

---

## Step 7 — Failure Analysis & Acceptance

> **Evaluation is valuable only when failures can be attributed to a specific stage and translated into a bounded engineering change.**

失败应该被归因到：

```text
Source
Chunking
Embedding
Retrieval Ranking
Context Construction
Prompt Semantics
Generation
Evaluation Label
```

而不是简单归因：

```text
“LLM 不够好”
```

Release Gate 应依据预定义指标和人工语义检查，而不是单个漂亮的 Demo。

---

# Day 6 Core Mental Model

> **RAG evaluation separates retrieval quality from generation quality and measures both against predefined, evidence-backed ground truth.**

Evaluation 的核心链路：

**Realistic Queries**

↓

**Ground Truth**

↓

**Retriever Output**

↓

**Retrieval Metrics**

↓

**Generated Answer**

↓

**Correctness + Groundedness + Refusal**

↓

**Failure Attribution**

核心公式：

**Good RAG = Good Retrieval × Good Generation**

而不是：

**Good RAG = High Similarity Score**

最终原则：

**Define First → Run → Measure → Diagnose → Improve**

而不是：

**Run → Look at Answers → Redefine Success**

---

# Day 7 — RAG Hardening & Production Readiness: One-Sentence Notes

## Step 1 — Scope Freeze

> **Before hardening a release candidate, freeze the architecture and acceptance scope so failures lead to bounded fixes rather than uncontrolled redesign.**

Day 7 不再随意加入新架构，而是固定：

**Retriever + Context Builder + Prompt Builder + DeepSeek + NDJSON**

之后只修复能够被证据证明的问题。

---

## Step 2 — Source Integrity Repair

> **When evaluation exposes a factual defect, repair the authoritative source and rebuild downstream artifacts instead of patching generated documents, chunks, or answers.**

例如参数标签错误必须从：

**Knowledge Source**

开始修复，然后重新经过：

**Source → Documents → Chunks → Vectors**

不能直接修改 Chunk 或 Prompt 来掩盖 Source Error。

---

## Step 3 — Knowledge Update Workflow

> **A production knowledge update should build and validate the complete snapshot before activation, while reusing unchanged embeddings only when their identity, content, and embedding configuration still match.**

统一 workflow：

**Source**

↓

**Staged Documents**

↓

**Staged Chunks**

↓

**Embedding Plan**

↓

**Reuse / Re-embed**

↓

**Complete Snapshot Validation**

↓

**Activation**

核心 reuse rule：

**Unchanged → Reuse**

**Changed → Re-embed**

**New → Embed**

**Deleted → Remove**

---

## Step 4 — Regression, Latency & Prompt Semantics

> **Hardening should distinguish retrieval defects from prompt-policy defects and fix the smallest layer responsible for the observed failure.**

Day 7 发现有些失败不是 Retrieval 问题，而是 LLM 对业务语义的错误推断，例如：

**Product Exists ≠ Manufacturer**

**Product Exists ≠ Supplier**

**Product Exists ≠ Stock Available**

因此需要修复 Prompt Semantics，而不是错误地修改 Chunking 或 Retrieval。

同时确认：

**Retrieval latency ≈ small**

**LLM generation remains the dominant request cost**

---

## Step 5A — New Holdout Design

> **A new holdout should contain genuinely unseen, independently authored cases that test the frozen system’s boundaries rather than repeating previously optimized examples.**

Holdout 应在系统冻结后建立，并覆盖：

* Product parameters
* Cross-field reasoning
* Recommendation
* Solution knowledge
* Company facts
* Closed-world facts
* Dynamic unknowns
* Abstention boundaries

一旦运行：

**Holdout → Seen Evidence**

之后不能再把它当作真正 Unseen Test Set。

---

## Step 5B — Sealed Holdout Evaluation

> **A sealed holdout is valuable only when it is run once against the frozen candidate and its original results are preserved without rerunning until they look better.**

Day 7 的 Holdout 原始 rubric 结果必须保留。

Owner 后续可以做 Business Acceptance Review，但不能：

```text
看到失败
→ 改 Prompt
→ 重跑同一 Holdout
→ 宣称仍然 Unseen
```

核心原则：

**Historical Result ≠ Retroactively Rewritten Result**

---

## Step 6 — Release Candidate Smoke

> **A release candidate is production-ready only when code, knowledge artifacts, retrieval behavior, API contracts, streaming, error handling, and representative end-to-end cases all pass together.**

最终 Smoke 覆盖：

**Tests**

**Snapshot Integrity**

**Retriever**

**Prompt**

**Generation**

**HTTP Contract**

**NDJSON Streaming**

**Request ID**

**Unknown Refusal**

**Latency**

只有完整链路通过，才能从：

**Development Candidate**

升级为：

**Release Candidate**

---

# Day 7 Core Mental Model

> **Production hardening turns a working RAG prototype into an evidence-backed release candidate by freezing scope, repairing root causes, validating the full knowledge lifecycle, and testing the complete system under realistic boundaries.**

Day 7 的核心过程：

**Freeze**

↓

**Find Failure**

↓

**Locate Root Cause**

↓

**Fix the Correct Layer**

↓

**Rebuild / Validate**

↓

**Regression**

↓

**Unseen Holdout**

↓

**Release Smoke**

↓

**Release Candidate**

核心原则：

**Source Error → Fix Source**

**Retrieval Error → Fix Retrieval**

**Prompt Semantics Error → Fix Prompt**

**Evaluation Error → Fix Evaluation**

不要：

**One Failure → Redesign Everything**

最终 Production Knowledge Lifecycle：

**Authoritative Source**

↓

**Validated Documents**

↓

**Validated Chunks**

↓

**Validated Vectors**

↓

**Validated Retrieval**

↓

**Grounded Generation**

↓

**Production Deployment**

Day 7 的最终目标不是“让所有测试看起来通过”，而是：

> **Know exactly what the system can do, what it cannot do, why it fails, and whether the tested release is safe enough to deploy.**

最终状态：

**RAG V1.1 → Evidence-Backed Release Candidate → Production**

