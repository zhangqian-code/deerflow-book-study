# 第 2 章 仓库全景与技术栈

这一章的目标不是记住每个文件名，而是先建立一张足够准确的“项目地图”。对当前 DeerFlow 2.0 来说，最重要的认知有三点：

1. 它已经不是一个把所有 Python 代码都塞在 `backend/src/` 里的单体后端。
2. 它的核心不是传统 Web 应用，而是一个以 Agent runtime 为中心的系统。
3. 当前代码已经明显分成两层：`App` 层和 `Harness` 层。

本章按当前仓库结构与实现说明 DeerFlow 的整体布局。

## 2.1 目录结构

当前仓库可以先粗分为 6 个部分：

```text
deer-flow/
├── backend/                         # Python 后端
│   ├── app/                         # 应用层：Gateway API、IM channels
│   ├── packages/harness/deerflow/   # 框架层：agent/runtime/tools/sandbox/skills
│   ├── tests/                       # 后端测试
│   ├── docs/                        # 后端设计与说明文档
│   ├── pyproject.toml               # backend workspace 入口
│   └── langgraph.json               # LangGraph server 配置
├── frontend/                        # Next.js 前端
├── skills/                          # Skills 目录（public/custom）
├── docker/                          # Docker 与 nginx 配置
├── scripts/                         # 统一启动、检查、部署脚本
├── config.example.yaml              # 主配置模板
├── extensions_config.example.json   # MCP/skills 状态模板
└── Makefile                         # 本地与 Docker 启动入口
```

如果你只想先快速理解当前项目，下面这两个目录最关键：

### `backend/app/`：应用层

这一层负责“把 DeerFlow 暴露给用户”。主要包括：

- `gateway/`：FastAPI Gateway，提供模型、MCP、skills、memory、uploads、artifacts、threads、runs 等 HTTP 接口
- `channels/`：飞书、Slack、Telegram、企微等 IM 渠道集成

这一层更像“产品壳”，回答的是“如何把 agent 能力通过 Web/API/IM 提供出去”。

### `backend/packages/harness/deerflow/`：框架层

这一层负责“Agent 本身如何构建和运行”。主要包括：

- `agents/`：Lead Agent、middleware、memory、checkpointer、thread state
- `sandbox/`：沙箱抽象、本地 provider、工具桥接
- `subagents/`：子 agent 注册与执行
- `tools/`：内置工具和工具装配
- `mcp/`：MCP 集成
- `skills/`：skills 发现、解析、加载、安装
- `models/`：模型工厂与 provider 适配
- `community/`：Tavily、Jina、Firecrawl、AIO Sandbox 等社区扩展
- `runtime/`：运行时、流桥、run 管理、状态序列化
- `config/`：主配置、扩展配置、路径与相关 schema
- `client.py`：内嵌 Python client

这一层回答的是“agent 怎么运行、怎么拿工具、怎么拿上下文、怎么进 sandbox、怎么调用 subagent”。

这里要特别注意一个很容易误解的点：

- 当前仓库**没有**顶层的 `sandbox/` 目录
- `sandbox` 是 `backend/packages/harness/deerflow/` 下面的一个子模块
- AIO Sandbox 的扩展实现则放在 `backend/packages/harness/deerflow/community/aio_sandbox/`

也就是说，当前项目里“Sandbox 是一个核心子系统”这句话是对的，但它在文件结构上属于 Harness 层，而不是仓库根目录下的独立模块。

## 2.2 App + Harness

理解 DeerFlow，最关键的不是背目录，而是先理解这套分层的意图。

### 为什么会拆成两层

早期后端更接近一个单体 Python 包。随着能力变多，这会带来几个问题：

- agent runtime 和产品接口混在一起，职责边界不清楚
- 复用 agent 能力时，必须连 FastAPI、IM SDK 一起带上
- 依赖越来越重，不利于把 agent 核心单独作为框架使用

因此，当前仓库开始把后端拆成：

- **Harness**：可复用的 agent 框架层
- **App**：面向 DeerFlow 产品本身的应用层

这不是单纯的目录美化，而是代码边界的重新定义。

### 当前应该怎么理解这两层

可以把 DeerFlow 想成两部分组合：

```text
App（backend/app）
  ├─ FastAPI Gateway
  ├─ IM channels
  └─ 调用 deerflow-harness

Harness（backend/packages/harness/deerflow）
  ├─ Lead Agent
  ├─ Middleware pipeline
  ├─ Tools / MCP / Skills
  ├─ Sandbox
  ├─ Subagents
  ├─ Memory / Checkpointer
  └─ Runtime
```

一句话概括：

- `App` 决定“如何对外提供能力”
- `Harness` 决定“能力本身如何运行”

这也是你后面看代码时最重要的分界线。

## 2.3 核心依赖：不要只看一个 `pyproject.toml`

学习当前仓库时，不要只看一个 `backend/pyproject.toml`，因为依赖被分布在 workspace 入口和 harness 包两层。

### `backend/pyproject.toml`：workspace / app 入口

这个文件现在更像 backend 工作区入口，里面直接声明：

- `deerflow-harness`
- `fastapi`
- `uvicorn`
- `sse-starlette`
- `python-multipart`
- `slack-sdk`
- `python-telegram-bot`
- `lark-oapi`

也就是说，它更偏向“应用层依赖 + 把 harness 拉进来”。

### `backend/packages/harness/pyproject.toml`：agent 核心依赖

真正与 agent runtime 强相关的依赖主要在这里，例如：

- `langgraph`
- `langgraph-api`
- `langgraph-runtime-inmem`
- `langgraph-checkpoint-sqlite`
- `langchain`
- `langchain-openai`
- `langchain-anthropic`
- `langchain-deepseek`
- `langchain-google-genai`
- `langchain-mcp-adapters`
- `agent-sandbox`
- `kubernetes`
- `tavily-python`
- `firecrawl-py`
- `ddgs`
- `duckdb`
- `markitdown`

因此，当前学习时更准确的说法是：

- `backend/pyproject.toml` 代表 backend workspace 和 app 依赖入口
- `backend/packages/harness/pyproject.toml` 代表 DeerFlow agent framework 的核心依赖

## 2.4 前端技术栈

前端仍然是一个相对独立的 Next.js 应用，这部分大方向没有变。

### 当前前端的基础栈

- **Next.js 16**
- **React 19**
- **TypeScript**
- **pnpm**
- **TanStack Query**
- **LangGraph SDK**

它的职责主要是：

- 提供聊天 UI
- 展示线程、消息、工具调用和产物
- 通过 Nginx 代理访问 Gateway 与 LangGraph 接口

需要注意的是，当前前端不是一个 monorepo 下的 pnpm workspace 多包结构；它更接近一个单独的 `frontend/` 应用。

## 2.5 配置体系

当前项目使用三层配置：

### 第一层：`config.yaml`

主配置文件，定义：

- `models`
- `tool_groups`
- `tools`
- `sandbox`
- `skills`
- `title`
- `summarization`
- `memory`
- `checkpointer`
- `subagents`
- `channels`

它决定的是“系统如何运行”。

### 第二层：`extensions_config.json`

用于保存扩展状态，主要包括：

- `mcpServers`
- `skills`

它决定的是“哪些扩展被启用”。

### 第三层：`.env`

用于放敏感配置与 API keys，并通过 `config.yaml` 中的 `$ENV_VAR` 语法引用。

因此，最简洁的理解方式是：

- `config.yaml`：运行参数
- `extensions_config.json`：扩展开关
- `.env`：密钥与敏感环境变量

## 2.6 支持的模型

DeerFlow 仍然是 model-agnostic 的，只要能通过配置适配到当前模型工厂，就可以接入。

当前仓库常见的模型接入方式包括：

| 类型 | 典型类路径 | 说明 |
|------|-----------|------|
| OpenAI / OpenAI 兼容 | `langchain_openai:ChatOpenAI` | 最常见，OpenRouter/兼容网关也常走这一层 |
| Anthropic | `langchain_anthropic:ChatAnthropic` 或封装类 | Claude 模型 |
| Gemini 原生 SDK | `langchain_google_genai:ChatGoogleGenerativeAI` | Gemini 原生接入 |
| DeepSeek / Kimi / 部分兼容模型 | `deerflow.models.patched_deepseek:PatchedChatDeepSeek` | 支持 reasoning / thinking 相关兼容 |
| 某些 OpenAI 兼容推理模型 | `deerflow.models.vllm_provider:VllmChatModel` 等 | 用于特定兼容场景 |

这里要特别注意：项目里的类路径以 `deerflow.models...` 为主。

## 2.7 运行时架构

当前默认运行模式仍然可以理解为 4 个服务协作：

```text
浏览器
  │
  ▼
Nginx :2026
  ├─ /api/langgraph/*  → LangGraph Server :2024
  ├─ /api/*            → Gateway API :8001
  └─ /*                → Frontend :3000
```

这和你在项目里运行 `make dev` 时看到的整体结构是一致的。

### Gateway mode

当前 DeerFlow 还支持 **Gateway mode**：

- 不单独启动 LangGraph Server
- 由 Gateway 内嵌 agent runtime
- 前端会改走 `/api/langgraph-compat`

也就是说，“4 服务架构”仍然是理解默认模式的好起点，但它已经不是唯一运行方式。

## 2.8 当前更准确的请求流

Gateway 并不是通过统一的 `/api/gateway/*` 前缀对外暴露。

当前更准确的理解是：

- `/api/langgraph/*` 走 LangGraph Server
- `/api/models`、`/api/mcp`、`/api/skills`、`/api/memory`、`/api/agents`、`/api/threads/*` 等走 Gateway
- `/` 走 Frontend

在默认模式下：

- LangGraph 负责 agent 交互、thread、streaming
- Gateway 负责模型、MCP、skills、memory、uploads、artifacts、threads、agents、channels、suggestions 等接口

在 Gateway mode 下：

- 一部分 LangGraph 兼容接口会由 Gateway 承担

## 2.9 这一章真正该记住什么

读完这一章，最值得带走的是下面这几个事实：

1. DeerFlow 当前是一个 **App + Harness** 分层的 agent 系统。
2. `backend/app` 是产品外壳，`backend/packages/harness/deerflow` 是 agent 核心。
3. 默认运行时由 `Nginx + Frontend + Gateway + LangGraph` 组成，但现在还支持 `Gateway mode`。
4. 配置体系仍然是 `config.yaml + extensions_config.json + .env` 三层。
5. skills 仍然是项目非常重要的扩展机制，而且物理上独立于 backend 代码。
6. `sandbox` 是 Harness 内部模块，不是仓库顶层目录；查代码时应该去 `backend/packages/harness/deerflow/sandbox/` 和 `backend/packages/harness/deerflow/community/aio_sandbox/`。

## 小结

- **目录结构**：后端分成 `App` 与 `Harness` 两层。
- **依赖分布**：核心 agent 依赖主要位于 `deerflow-harness`，同时 backend workspace 负责应用层依赖与工作区入口。
- **运行时**：除了默认的四服务模式，项目还支持 Gateway 内嵌 runtime 的运行方式。
- **学习建议**：后面读源码时，优先用“App 提供外壳，Harness 负责 agent 核心”这个视角去看，理解会快很多。
