# AI Agent 岗位项目学习计划

> 目标岗位：华为 AI Agent 开发岗（实习/校招）
> 适用人群：大三、有编程语言基础但暂无项目经验
> 核心思路：用一个大项目把所有考点串起来，学什么都是为了把它做深，而不是堆十几个教程 demo。

---

## 一、先定技术选型

| 类别 | 推荐 | 备注 |
| --- | --- | --- |
| 语言 | Python | LLM/AI 生态最快，`FastAPI` + `Pydantic` + `asyncio` 足够做后端工程 |
| 编排 | 先手写 Agent Loop，再用 `LangGraph` | 不建议一上来就堆 LangChain 全家桶 |
| 可观测性 | `structlog` + `Langfuse` / `OpenTelemetry` | 复盘每次 Agent 运行 |
| MCP | 官方 `mcp` Python SDK（含 `FastMCP`） | stdio / streamable HTTP |
| RAG 解析 | `PyMuPDF` / `unstructured` | PDF/DOCX/Markdown、表格 |
| 切分 | `tiktoken` | 按 token 而不是字符切 |
| Embedding | OpenAI embedding 或本地 `BGE-M3` | 本地模型省成本、可离线 |
| 向量库 | `Chroma` / `Qdrant` | 入门用 Chroma，进阶换 Qdrant/Milvus |
| 混合检索 | `rank-bm25`（BM25）+ 向量 + RRF 融合 | 关键词 + 语义互补 |
| Rerank | `BGE-reranker` 或 Cohere Rerank | cross-encoder 精排 |
| 评估 | `pytest` + 自建评测集，可选 `ragas` | 回归测试 + 指标对比 |

---

## 二、分阶段计划

### 阶段 0：补齐工程地基（1–2 周）
- 技术栈：`Python` 进阶（类型标注、异常、装饰器、上下文管理器、`asyncio`）、`Git`、HTTP/JSON/`JSON Schema`、`FastAPI` + `httpx`。
- 产出：写一个带参数校验和错误码的 REST API，自己调通，提交到 GitHub。
- 对应考点：后端语言基础、工程代码能力。

### 阶段 1：LLM 基础认知 + 第一次可控调用（1–2 周）
- 技术栈：`openai` SDK（流式、`tool calling`、结构化输出）、`tiktoken`、`JSON Schema`。
- 学透三个概念并落到代码：
  - `Token`：怎么估算用量、超长怎么发现。
  - `Context Window`：输入 + 输出 + 工具结果都要算。
  - `Temperature`：什么时候要低、结构化输出要强制。
- 产出：一个 CLI，能流式调用、打印 token 用量、强制 JSON 输出、处理超时/限流/重试。
- 对应考点：理解这些概念在实际开发中意味着什么。

### 阶段 2：最小工具调用循环（2–3 周）
- 技术栈：`function calling` 全链路（工具 Schema → 模型返回 `tool_call` → 执行 → 结果回填）。
- 手写 Agent Loop，重点做三件事：
  - 参数校验；
  - 工具执行失败怎么处理；
  - 循环终止条件（最大步数 / 达到目标 / 模型主动结束）。
- 产出：一个能查天气、算日期、记笔记的最小 Agent。
- 对应考点：Agent Loop 的终止条件、工具调用失败处理。

### 阶段 3：Agent 工程化——状态机、错误恢复、可观测性（3–4 周，最核心）
- 技术栈：显式状态机 / `LangGraph`（`StateGraph` + checkpoint）、重试与指数退避、消息裁剪与 token 预算、`structlog` + `Langfuse` trace。
- 要处理的边界问题：
  - 上下文溢出（裁剪/摘要/预算控制）；
  - 工具返回非法结果；
  - 模型输出不符合预期（用 Schema 拦截）。
- 产出：把阶段 2 改成显式状态机，加入超时、重试、非法调用拦截、每次运行可复盘日志。
- 对应考点：状态管理、异常恢复、上下文溢出、可观测性。

### 阶段 4：MCP（2–3 周）
- 技术栈：官方 `mcp` SDK（`FastMCP`）、stdio / streamable HTTP、工具白名单与权限边界。
- 产出：自己写 1 个 MCP Server 暴露 2–3 个工具 + 1 个 Client，让阶段 3 的 Agent 通过 MCP 调用；再接入一个官方现成 server 作对比。
- 对应考点：MCP Server/Client 实现、安全可控地操作真实环境。

### 阶段 5：RAG Pipeline（3–4 周）
- 技术栈：文档解析（PDF/DOCX/Markdown、表格）、chunking（固定长度 vs 语义切分、重叠、层级）、embedding、向量检索、混合检索（BM25 + 向量 + RRF 融合）、rerank（cross-encoder）、引用溯源（chunk 元数据定位原文）。
- 产出：一个可复现的 pipeline，对同一语料做 3 种 chunking 策略对比，输出召回率；回答带原文引用。
- 对应考点：chunking 策略、检索方案、召回率评估、引用溯源。

### 阶段 6：评估体系（2–3 周）
- 技术栈：评测集（query + 标准答案/参考文档）、指标（检索召回/精度、答案准确率、引用正确率）、`pytest` 回归测试、prompt 版本管理与 A/B 对比。
- 产出：30–50 条评测集，一键跑分；改 chunking/rerank/prompt 前后能对比分数。
- 对应考点：评估体系搭建、prompt 的工程化管理/测试/迭代。

### 阶段 7：生产级加固 + GitHub 展示（持续）
- 技术栈：配置管理、单元/集成测试、README + 架构图 + 设计文档、复盘笔记。
- 产出：仓库 README 有架构图、可一键运行、设计文档写清“为什么这么选、踩了什么坑、trade-off 是什么”。
- 对应考点：GitHub 真实项目、能深入聊的项目。

---

## 三、旗舰项目怎么设计

建议做一个「本地知识库问答 + 工具调用 Agent」，一个项目同时覆盖 Agent Loop、MCP、RAG、评估、可观测性：

- M1：RAG 检索一段语料（课程笔记或一组 PDF），回答带引用。
- M2：加上工具调用（文件搜索、计算器、日期、知识库查询）。
- M3：用 MCP 把工具包起来，改成显式状态机 + 重试 + 日志 trace。
- M4：建评测集，量化 chunking/rerank/prompt 改动的效果。
- M5：写设计文档 + 复盘，GitHub 提交历史干净。

比“什么都做一点”更重要的，是你能回答出每个决策的原因：为什么用混合检索而不是纯向量、为什么选这个 chunk 大小、工具失败时是重试还是降级。

---

## 四、面试怎么准备

对方考察的是“讲清楚设计决策和 trade-off”。每个模块做完都写一段复盘，固定用这个框架：

> 背景与目标 → 方案选择 → 为什么不用别的方案 → 踩了什么坑 → 用什么指标证明它更好。

这比多学一个框架值钱得多。重点准备好以下三个问题的现场推演：

- Agent 跑飞了怎么办？
- 工具连续失败怎么办？
- 上下文爆了怎么办？

---

## 五、时间与避坑

- 按每周 10–15 小时算，主线跑通（阶段 0–5）约 3–4 个月，后面持续打磨。
- 以你下一次暑期实习/日常实习投递为锚点倒排：春季投递前至少要把阶段 3 和 4 跑通。
- 避坑清单：
  - 不要只跑教程 demo；
  - 不要堆 LangChain/LlamaIndex 全家桶；
  - 不要追求最新模型；
  - 不要忽略日志和测试。
