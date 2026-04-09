# 第 1 章 DeerFlow 是什么

这一章的目标很简单：先回答一个问题。

**DeerFlow 到底是什么样的系统？**

如果这个问题没有想清楚，后面看目录、看启动链路、看 Agent 代码时，很容易把它误解成普通 Web 应用，或者误解成一个“带聊天界面的 LLM Demo”。当前仓库里的 DeerFlow 不是这两种东西。

## 1.1 一句话定义

DeerFlow 是一个开源的 **super agent harness**。

“super agent harness” 这个说法可以拆成两部分理解：

- **agent**：它的核心是一个会思考、会调用工具、会拆任务、会产出结果的智能体系统
- **harness**：它不只是提示词和模型调用，而是把运行时真正需要的基础设施一起组织好了

这些基础设施包括：

- sub-agents
- memory
- sandbox
- skills
- MCP
- tools
- runtime

所以，DeerFlow 不是单纯的“提示词工程项目”，也不是只负责模型编排的轻量库。它更接近一个让 agent 真正能持续完成复杂任务的运行平台。

## 1.2 它解决的是什么问题

很多 LLM 应用只解决“如何问模型”这个问题，但实际复杂任务往往还需要解决下面这些问题：

### 任务会持续很久

不是一问一答，而是几分钟、十几分钟，甚至更长时间的执行流程。

### 任务需要访问外部能力

例如：

- 搜索网页
- 读取文件
- 写文件
- 调用命令
- 连接第三方工具
- 调用子 agent

### 任务过程中要保持上下文

系统需要知道：

- 当前线程里已经发生过什么
- 用户上传了哪些文件
- 已经生成了哪些产物
- 哪些信息应该被记忆
- 哪些上下文应该被压缩或保留

### 任务最终要产出交付物

很多时候，目标不是一句回答，而是：

- 报告
- 文档
- 网站
- 幻灯片
- 分析结果
- 代码或代码修改方案

DeerFlow 的价值就在于：它把这些问题放在同一个系统里统一处理。

## 1.3 为什么叫 Harness

如果把一般框架看成“给你零件”，那 Harness 更像“把关键零件和连接方式都先搭起来”。

在当前 DeerFlow 里，系统已经预设了很多核心约定：

- 有一个 Lead Agent 作为主控入口
- 有 middleware pipeline 处理线程数据、sandbox、memory、title、clarification 等横切逻辑
- 有 skills 机制，把领域能力以结构化方式注入 agent
- 有 sandbox，把文件、命令和执行环境隔离起来
- 有 subagents，支持把复杂任务拆出去并行执行
- 有 Gateway、Frontend、Nginx，形成完整的交互入口

这意味着 DeerFlow 是一个**带明确架构立场**的系统。

你当然可以替换其中的某些部分，但默认情况下，它已经给出了一套“足够完整、能直接工作的 agent 运行方式”。

## 1.4 DeerFlow 面向什么任务

DeerFlow 最擅长的是 **long-horizon tasks**，也就是需要持续执行、逐步决策、最后产出结果的任务。

这类任务通常有几个共同点：

### 不是一步完成

模型不会一次调用就结束，而是需要多轮思考、工具调用、继续思考。

### 任务结构不是固定的

系统不会总是走相同流程。不同请求可能触发不同 skills、不同工具、不同 subagents。

### 需要处理真实工作对象

真实工作对象包括：

- 文件
- 目录
- 网页
- 图片
- 线程状态
- 外部服务

### 最后要有“能交付”的结果

例如一份研究报告、一个分析文档、一个生成的网站或一个工作流结果。

所以，理解 DeerFlow 时，最好把它看成：

**一个围绕“复杂任务执行”而构建的 agent 系统。**

## 1.5 当前仓库里的核心能力

从当前代码来看，DeerFlow 的核心能力至少包括下面这些模块。

### Lead Agent

Lead Agent 是系统的主入口。用户任务进入系统后，最终都会落到这个主控 agent 上，由它决定：

- 怎么理解任务
- 是否需要澄清
- 是否加载某个 skill
- 是否调用工具
- 是否拆给 subagent
- 最终如何组织结果

### Middleware Pipeline

很多看起来不像“Agent 推理”的事情，其实是通过 middleware 完成的，例如：

- 创建 thread 对应的目录
- 注入上传文件
- 获取 sandbox
- 做上下文摘要
- 生成标题
- 更新 memory
- 处理图片查看
- 触发 clarification

这使得 DeerFlow 的 agent 不是裸的模型调用，而是“模型 + 中间件 + 工具 + 状态”的组合体。

### Sandbox

Sandbox 负责代码与文件操作环境。

它不是仓库根目录下的一个独立文件夹，而是 Harness 层中的核心模块。当前你主要会在这些位置看到它：

- `backend/packages/harness/deerflow/sandbox/`
- `backend/packages/harness/deerflow/community/aio_sandbox/`

它的作用包括：

- 提供受控的文件系统视图
- 提供命令执行能力
- 管理虚拟路径与真实路径映射
- 为不同线程提供隔离的数据目录

### Skills

Skills 是 DeerFlow 非常有辨识度的一层。

它们以目录和 `SKILL.md` 的形式存在于仓库中，物理上独立于 Python 后端代码。当前技能目录在：

- `skills/public/`
- `skills/custom/`

skills 不是一个单独工具调用，而是把“完成某类任务的方法、流程、规则、参考资料”打包给 agent。

### Subagents

Subagents 让系统可以把复杂任务拆出去并行完成。

这意味着 DeerFlow 的执行模型不只是“一个 agent 连续思考”，而是“一个主 agent 可以临时调度其他 agent 协作完成工作”。

### Memory

Memory 让 DeerFlow 能在多轮和多任务中保留更稳定的用户与任务上下文。

这层能力的重点不是单纯存聊天记录，而是：

- 抽取用户相关事实
- 组织成结构化记忆
- 在后续任务中重新注入

### MCP 与外部能力接入

DeerFlow 可以通过 MCP 接入更多外部能力，这让 agent 不必只依赖内置工具。

当前仓库中，MCP 是扩展体系的重要组成部分，也是在理解 DeerFlow“为什么能扩展成复杂工作平台”时必须关注的一层。

## 1.6 从学习角度，应该把 DeerFlow 看成什么

如果你的目标是读懂当前仓库，最推荐的视角不是“产品功能列表”，而是下面这个结构：

```text
用户任务
  ↓
Lead Agent
  ↓
Middleware / State / Runtime
  ↓
Tools / Skills / MCP / Sandbox / Subagents
  ↓
文件、网页、命令、外部服务、最终产物
```

这个视角有两个好处：

1. 你会知道为什么后面要先看仓库结构、启动方式、配置体系
2. 你会知道为什么 DeerFlow 不能只用“提示词工程”来理解

它真正的复杂度不在某一个 prompt，而在整套运行时是如何组织起来的。

## 1.7 DeerFlow 在当前仓库中的形态

从当前代码组织方式看，DeerFlow 已经明显分成两层：

### App 层

位置主要在：

- `backend/app/`

负责：

- Gateway API
- IM 渠道
- 对外 HTTP 入口

### Harness 层

位置主要在：

- `backend/packages/harness/deerflow/`

负责：

- agent runtime
- sandbox
- tools
- skills
- models
- subagents
- memory
- config
- runtime

因此，你后面看源码时可以始终带着一个最有用的问题：

**这一段代码是在做“产品壳”的事，还是在做“agent 核心运行时”的事？**

这个判断会极大降低理解难度。

## 1.8 这一章应该记住什么

读完这一章，建议先记住下面这几件事：

1. DeerFlow 是一个 super agent harness，不是普通聊天应用。
2. 它关注的是复杂任务执行，不只是一次性问答。
3. 它的核心不是单个 prompt，而是 Agent + Runtime + Tools + Sandbox + Skills + Memory 的整体。
4. 当前仓库已经分成 `App` 与 `Harness` 两层，这会影响你后面对代码的理解方式。
5. 你后面学习时，应该始终围绕“任务是如何被接收、组织、执行、产出”的主线去看。

## 小结

- **DeerFlow 的本质**：一个让 agent 能长期执行复杂任务的运行平台
- **核心能力**：Lead Agent、middleware、sandbox、skills、subagents、memory、MCP
- **核心理解方式**：把它看成“任务执行系统”，而不是“聊天界面”
- **阅读源码的主线**：任务入口 → agent runtime → tools / sandbox / skills / subagents → 产物输出
