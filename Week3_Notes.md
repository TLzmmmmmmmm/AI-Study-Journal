# Day 1 — Tools & Deterministic Capabilities: One-Sentence Notes

## Step 1 — Tool Calling Mental Model

> **An LLM proposes a tool call, but only the backend validates and executes an allowlisted capability.**

Tool Calling 的核心是 **LLM chooses; backend executes**：模型只能生成 Tool Name + Arguments，真正的执行权始终属于 Backend。

---

## Step 2 — RAG vs Tools

> **RAG retrieves semantically relevant evidence, while deterministic tools provide controlled access to exact facts, structured state, business rules, or actions.**

RAG 主要解决“哪些 Evidence 相关”，Tool 主要解决“系统可以可靠地做什么”；二者并不互斥，Tool 也可以把 RAG 封装成受控能力。

---

## Step 3 — Product Search vs Exact Lookup

> **`search_products()` returns ranked product candidates with evidence, while `get_product_details()` returns an authoritative product object for an exact canonical identity.**

`search_products()` 使用语义检索解决 Product Discovery；`get_product_details()` 使用 Exact Entity Resolution + Structured Source 解决精确产品事实查询。

---

## Step 4 — Tool Contract

> **A Tool Contract defines a capability boundary through its description, input schema, validation rules, output schema, errors, and provenance.**

Function 定义“代码怎么执行”，Tool Contract 定义“LLM 被允许如何使用这个能力”；只暴露完成任务所需的最小控制面。

---

## Step 5 — Tool Safety

> **Every LLM-generated tool call is untrusted input and must pass allowlisting, validation, normalization, business checks, and controlled execution.**

LLM 不能直接获得 Python、Shell 或任意函数执行权；Tool Layer 必须遵循 **least privilege + fail closed**，禁止 `eval`、`exec`、动态 `getattr` 等任意执行路径。

---

## Step 6 — Provenance vs Citation

> **Provenance is machine-readable source identity that travels with the data, while citation is the later user-facing rendering of that provenance.**

Tool Result 应在产生数据时携带可信 `SourceRef(title, url)`；LLM 不负责猜测 URL，Citation 只是后续把 Provenance 展示给用户。

---

## Step 7 — Tool Error Model

> **Stable Tool Error codes turn failures into explicit capability semantics instead of leaking implementation exceptions.**

Tool Layer 使用统一错误契约：

**INVALID_ARGUMENT · PRODUCT_NOT_FOUND · AMBIGUOUS_PRODUCT · TOOL_UNAVAILABLE · TOOL_EXECUTION_ERROR**

Backend 根据稳定 `code` 控制流程，而不是依赖可能变化的错误文字。

---

# Day 1 Core Mental Model

> **Tools turn backend capabilities into narrow, deterministic, validated interfaces that an LLM may request but never directly execute.**

核心分工：

**LLM = Choose**

**Backend = Validate & Control**

**Tool = Execute**

**Provenance = Trace**

最终原则：

**Semantic Search → RAG · Exact Facts / Rules / Actions → Deterministic Tools**

---

# Day 2 — Raw Agent Loop: One-Sentence Notes

## Step 1 — Workflow, Orchestration & Agent

> **A workflow defines the execution structure, orchestration coordinates how it runs, and an Agent introduces bounded LLM decisions into that workflow.**

原本 RAG 已经是 deterministic workflow；Agent 并不是替代 Workflow，而是在其中加入 **Decide → Act → Observe → Decide Again** 的动态决策闭环。

---

## Step 2 — Agent State & Message Flow

> **Agent continuity is created by carrying actions and observations forward in state before every new LLM decision.**

LLM 不会自动记住上一轮 Tool Execution；Backend 通过：

**assistant(tool_call) → tool(observation) → next LLM call**

把 Action 和 Observation 写回 `messages`，让下一次决策看到刚才真实发生了什么。

---

## Step 3 — Agent Loop & Termination

> **The LLM chooses the next action, but the backend owns the Agent’s global limits and termination conditions.**

Agent autonomy 必须同时受到 **Action Bound + Time Bound** 控制：

**MAX_TOOL_CALLS + LLM Timeout + Tool Timeout + Overall Agent Timeout**

达到 Tool 上限后，通过 tools-disabled final completion 强制生成最终答案。

---

## Step 4 — Error as Observation & Self-Correction

> **A Tool Error can become a structured observation that lets the Agent correct, clarify, or safely fallback instead of immediately terminating.**

Tool-level failure可以形成：

**Bad Action → Error Observation → New Decision**

这不同于机械 Retry，因为 Agent 可以根据失败原因改变下一次 Action；但 self-correction 始终受到 Tool Call 和时间预算限制。

---

## Step 5 — Multi-turn Context, RAG & Agent

> **Conversation history explains what the user means, RAG retrieves relevant knowledge, Tool observations report runtime results, and the Agent decides what to do next.**

四类 Context 各司其职：

**System = Rules**

**History = Conversation Understanding**

**RAG = Retrieved Evidence**

**Tool Observation = Action Result**

Conversation History 可以帮助 LLM 理解“它 = LY198”，但当前 Retriever 仍只使用最后一条原始 User Message，以保持稳定、可评测的 Retrieval Boundary，不进行 Query Rewriting。

---

# Day 2 Core Mental Model

> **An Agent is a bounded system that uses an LLM to repeatedly decide, act through controlled tools, observe results, and decide again until it can safely answer.**

最小 Agent：

**Agent = LLM + State + Tools + Policies + Orchestration**

核心循环：

**Decide → Act → Observe → Update State → Decide Again → Terminate**

最终分工：

**LLM = Decide**

**Orchestrator = Coordinate**

**Tool = Execute**

**Observation = Feedback**

**Backend = Bound & Terminate**
