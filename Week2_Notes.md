```markdown
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
```

```markdown
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
```
