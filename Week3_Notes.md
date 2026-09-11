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

# Day 3 — Routing & Hybrid Orchestration: One-Sentence Notes

## Step 1 — Routing Fundamentals

> **A Router chooses the execution strategy for a request, while orchestration coordinates how that selected strategy actually runs.**

Router 解决“这次请求应该走哪条能力路径”，Orchestrator 负责真正执行；Routing 的核心原则是 **deterministic when possible, agentic when necessary**。

---

## Step 2 — Capability Classification & Route Taxonomy

> **Routes should represent the execution capability a request requires, not merely the keywords or topic it contains.**

Routing V1 使用六类稳定能力：

**product_search · exact_product · contact · knowledge · direct · fallback**

例如明确要求产品推荐即进入 `product_search`，而不是因为出现“酒店”“停车场”等场景词就自动进入 Knowledge RAG。

---

## Step 3 — Hybrid Router

> **A Hybrid Router resolves high-confidence cases deterministically and uses a constrained LLM classifier only when semantic intent remains ambiguous.**

执行顺序为：

**Deterministic Signals → unresolved? → Constrained LLM Router → Frozen Route Enum**

LLM Router 只负责分类，不执行 Tool，也不决定是否需要 Agent；**LLM classification ≠ agentic execution**。

---

## Step 4 — Deterministic Workflow vs Agentic Execution

> **If the execution path is known in advance, run it deterministically; use an Agent only when the next action depends on context, multiple capabilities, or observations.**

单一能力请求直接执行：

**exact_product → get_product_details**

**product_search → search_products**

**contact → get_contact_info**

**knowledge → RAG**

只有 mixed intent、上下文未解析或下一步依赖 Observation 时才进入 Agent Loop。

---

## Step 5 — Primary Route & Mixed Intent

> **A mixed request may have one primary route for initial execution while still escalating to an Agent when multiple capabilities are required.**

V1 不建立复杂 Multi-Route Planner，而是先确定主要 execution anchor：

**knowledge > product_search > exact_product > contact > fallback > direct**

Primary Route 决定起点，`agentic=True` 表示后续仍需要自适应调用其他安全能力。

---

## Step 6 — Agent Trace & Failure Attribution

> **A Trace records observable execution lineage without exposing conversation payloads, private data, or model reasoning.**

Route Trace 只保留：

**route · tool outcome · tool count · chunk IDs · failure layer**

Failure Layer 用于定位未恢复的系统失败：

**ROUTING · TOOL_SELECTION · ARGUMENT_GENERATION · TOOL_EXECUTION · RETRIEVAL · GENERATION**

中间错误如果被 Agent 成功纠正，只留在 attempt/tool trace 中，最终 `failure_layer=None`。

---

## Step 7 — Routing Evaluation

> **Routing quality is measured by whether each request reaches the correct capability with minimal unnecessary retrieval or agentic execution.**

Routing V1 重点关注：

**Route Accuracy · Unnecessary Retrieval Rate · Tool Selection Accuracy · Tool Execution Success · Fallback Accuracy**

Routing 的价值不仅是降低成本，更重要的是把请求送到正确的 Evidence 和 Capability Boundary。

---

# Day 3 Core Mental Model

> **Routing separates coarse execution-strategy selection from fine-grained adaptive Agent decisions, allowing known paths to stay deterministic while preserving Agent flexibility when runtime uncertainty appears.**

核心架构：

**User → Router → Execution Strategy → Orchestrator**

其中：

**Router = Choose Path**

**Orchestrator = Coordinate**

**Deterministic Workflow = Execute Known Path**

**Agent = Adapt to Observation**

最终原则：

**Path Known → Deterministic · Next Action Unknown → Agentic**

---

# Day 4 — Citation Pipeline & Grounded Generation: One-Sentence Notes

## Step 1 — Evidence, Provenance & Citation

> **Evidence supports a claim, provenance records where that evidence came from, and citation exposes that provenance to the user.**

三者不能混淆：

**Evidence = Supporting Content**

**Provenance = Source Identity**

**Citation = User-Facing Source**

真实 Citation 并不能自动让一个 unsupported answer 变成 Grounded Answer。

---

## Step 2 — SourceRef Contract & Ownership

> **`SourceRef(title, url)` is a minimal backend-owned provenance contract that travels with evidence without requiring the LLM to reconstruct source identity.**

RAG 与 Tool 都统一携带可信 `SourceRef`；`title` 和 `url` 均由 Backend 控制，LLM 不负责生成、翻译、猜测或重建来源。

---

## Step 3 — Grounded Answer Generation

> **A grounded answer keeps factual claims within the support of the evidence produced by the selected execution path.**

不同 Route 的事实边界不同：

**knowledge → RAG Context**

**exact_product → Product Tool Result**

**product_search → Candidate Evidence**

**contact → Contact Tool Result**

如果 Evidence 不足，应限制回答或明确无法确认，而不是让模型用常识补全公司事实。

---

## Step 4 — Citation Collection & Deduplication

> **The Citation Collector gathers request-scoped trusted SourceRefs, deduplicates them by URL identity, and preserves deterministic first-seen order.**

Collector 只负责：

**Collect → Validate → Normalize Identity → Deduplicate → Preserve Order**

不会重新 Retrieval、猜 URL、修改 title 或调用 LLM；相同 URL 保留第一次出现的可信 SourceRef。

---

## Step 5 — Route-Specific Citation Policy

> **Citation visibility should follow successful evidence-producing execution, not query keywords or naive refusal-text detection.**

正常事实回答：

**knowledge · exact_product · product_search · contact → normally cite**

而：

**direct · backend fallback · safe fallback → normally no citation**

Mixed Agent 汇总成功 RAG / Tool provenance 后统一去重；V1 暂不通过“无法确认”等关键词判断自由文本纯拒答。

---

## Step 6 — Citation Rendering & Streaming

> **The backend formats citations as presentation text while the transport layer remains responsible only for delivering `delta`, `done`, and `error` events.**

内部始终保持：

**answer text ≠ SourceRef[]**

最终响应边界才执行：

**Clean Answer + Backend References → complete delta → done**

Formatter 只返回纯文本，不生成 NDJSON；未来恢复 token streaming 后，可以把 Citation 作为最后一个 `delta` 再发送 `done`。

---

## Step 7 — Citation Integrity & Evaluation

> **Citation evaluation checks source integrity, relevance, completeness, deduplication, and whether unauthorized model-generated URLs can reach the user.**

V1 优先保证：

**Citation Presence Accuracy · URL Validity · Deduplication Accuracy · Route Policy Accuracy · Unauthorized URL Rate**

模型正文中的 URL 和自生成引用段由 Backend 清理，可信 URL 只能通过 Citation Formatter 输出。

Citation 问题属于独立的 Trust / Presentation Quality，不改变 Day 3 `failure_layer` 的语义。

---

# Day 4 Core Mental Model

> **Grounding constrains what the model may claim, provenance preserves where the evidence came from, and the backend turns trusted provenance into citations without giving the LLM authority over source identity.**

完整链路：

**Evidence → Grounded Answer**

**Evidence → SourceRef → Collector → Formatter → Citation**

核心分工：

**LLM = Answer Wording**

**RAG / Tool = Evidence**

**Backend = Source Authority**

**Collector = Deduplicate**

**Formatter = Present**

最终原则：

**LLM owns the answer text · Backend owns the sources · Citation follows evidence**

# Day 5 — LangGraph Orchestration & Production Migration: One-Sentence Notes

## Step 1 — LangGraph Mental Model

> **LangGraph makes orchestration explicit by representing a workflow as state transitions across nodes connected by edges instead of hiding control flow inside nested `if` statements and `while` loops.**

LangGraph 不替代 Router、RAG、Tool 或 LLM，而是把原本隐含在 Python Control Flow 中的 **When · What Next · Branch · Loop · Stop** 显式表示成 Graph。

---

## Step 2 — State vs Dependencies

> **Graph State stores request-scoped data that must survive across node boundaries, while dependencies are shared capabilities that nodes use to perform work.**

State 应只保存 Workflow 执行过程中需要跨 Node 传递的数据，例如：

**Messages · Route Decision · Retrieval Hits · Tool Call · Tool Traces · Pending Failures · Answer**

而：

**Router · Retriever · ToolExecutor · `complete_chat`**

属于 Dependencies，不应该放进 State。

没有 Reducer 时，Node 返回的是 **Partial State Update**，同名字段采用 overwrite；可变 `list` / `dict` 应使用 copy-on-write，而不是直接修改旧 State。

---

## Step 3 — Node, Edge & Conditional Edge

> **A Node performs one meaningful workflow step, an Edge defines a fixed transition, and a Conditional Edge chooses the next step from the current state.**

Graph 不应该把每个 Python Function 都变成 Node，而应围绕有独立 Workflow 意义的步骤建模：

**Route · Retrieve · Generate · Agent Step · Execute Tool · Finalize**

其中：

**Edge = Always Go Next**

**Conditional Edge = Inspect State → Choose Next Node**

Tool Observation 本身是 State Data，而不是独立 Node。

---

## Step 4 — Hybrid Routing as Graph Topology

> **Routing chooses the capability anchor, while `agentic` determines whether execution follows a fixed workflow or enters an adaptive Agent cycle.**

最终 Graph 保留 **deterministic when possible, agentic when necessary**：

**FALLBACK → always fallback**

**KNOWLEDGE → always retrieve first**

然后：

**KNOWLEDGE + non-agentic → RAG Generate**

**KNOWLEDGE + agentic → Agent**

其他 Route：

**Known Path → Deterministic Tool / Direct**

**Next Action Unknown → Agent**

因此 `Route` 和 `agentic` 是两个不同维度：

**Route = Where execution starts**

**Agentic = Whether execution must adapt to runtime observations**

---

## Step 5 — Agent Loop as a Graph Cycle

> **A Raw Agent `while` loop becomes an explicit graph cycle in which `agent_step` decides, `execute_tool` acts, and a conditional edge determines whether to loop or finalize.**

原本：

**Decide → Act → Observe → Decide Again**

从隐藏的 Python Loop：

**`while ...`**

变成显式 Graph：

**Agent Step → Tool? → Execute Tool → Agent Step**

Agent State 跨轮保存：

**Provider History · Processed Call Count · Request Cache · Tool Traces · Pending Failures · Successful Sources**

Backend 仍然控制 Tool Budget：

**最多 3 次 Tool Execution**

第三次 Observation 后再执行一次 **tools-disabled final completion**，模型不能执行第四次 Tool Call。

---

## Step 6 — Failure Recovery & Agent State

> **Agent failures are stateful unresolved conditions that may be recovered by later successful actions, so they cannot be represented by only the latest error.**

`pending_failures` 保存所有尚未被后续成功操作恢复的问题：

**TOOL_SELECTION**

**ARGUMENT_GENERATION**

**TOOL_EXECUTION + Tool Name**

成功 Tool Execution 可以：

**清除 TOOL_SELECTION**

**清除 ARGUMENT_GENERATION**

**清除同名 Tool 的 TOOL_EXECUTION**

因此：

**Single Tool Failure Classification ≠ Agent Failure State**

最终 Failure 优先级为：

**Terminal Explicit Failure → First Unresolved Pending Failure → None**

---

## Step 7 — Finalization & Production Parity

> **A production graph must preserve not only the final answer, but also source ordering, failure precedence, tool traces, retrieval IDs, and exception metadata.**

Agent 内部首先形成：

**Agent State → AgentResult**

然后 Orchestration 层形成：

**AgentResult + Route + Retrieval Evidence → RouteExecutionResult**

Non-Agentic 与 Agentic Finalization 保持独立，因为二者拥有不同的 Tool Trace、Failure Recovery 和 Evidence Aggregation 语义。

Migration 过程中不仅比较 Happy Path，还必须保持：

**Answer · Sources · RouteTrace · Failure Layer · Tool Calls · Retrieved IDs · Exception Metadata**

因此重构原则是：

**Behavior First · Abstraction Second**

---

## Step 8 — Production Takeover

> **LangGraph replaces orchestration control flow, not the underlying capabilities that perform routing, retrieval, tool execution, generation, citation, or transport.**

最终 Production Architecture：

**Request**

→ **GraphRouteOrchestrator**

→ **Compiled LangGraph**

→ **Router / Retriever / ToolExecutor / LLM**

→ **RouteExecutionResult**

→ **Citation & Sanitization**

→ **NDJSON Response**

Graph 在 Application Lifespan 中只 Compile 一次，并在多个请求之间复用。

Request-local 数据仍独立保存：

**AgentDeadline · Graph State · Tool Cache**

因此共享 Compiled Graph 不等于共享 Request State。

---

## Step 9 — LangGraph, Agent, RAG & MCP

> **LangGraph orchestrates workflows, an Agent provides adaptive decision-making, RAG supplies external evidence, and MCP standardizes how external capabilities and context can be exposed to AI systems.**

四者不是互相替代的技术：

**RAG = Retrieve Evidence**

**Tools = Execute Controlled Capabilities**

**Agent = Decide the Next Action**

**LangGraph = Orchestrate State & Transitions**

**MCP = Standardize External Capability / Context Interfaces**

当前项目使用 LangGraph 管理 Workflow，但没有必要为了使用 LangGraph 而同时引入 MCP。

MCP 是 Capability Integration Protocol，不是 Agent Loop 或 Graph Orchestration 的替代品。

---

# Day 5 Core Mental Model

> **LangGraph does not make the Agent smarter; it makes the system’s state, decisions, transitions, loops, and termination rules explicit, testable, and production-controllable.**

核心抽象：

**State = What the workflow remembers**

**Node = What happens now**

**Edge = What happens next**

**Conditional Edge = How the next path is chosen**

**Cycle = Bounded Agent Loop**

**Dependency = Capability used by Nodes**

完整架构：

**User**

→ **Router**

→ **Deterministic Workflow / RAG / Agent**

→ **Tools & Evidence**

→ **Finalization**

→ **RouteExecutionResult**

其中：

**Path Known → Deterministic**

**Knowledge Needed → RAG**

**Next Action Unknown → Agentic**

**Workflow Coordination → LangGraph**

最终原则：

**LangGraph owns orchestration · Components own capabilities · State carries runtime truth · Backend owns control**

