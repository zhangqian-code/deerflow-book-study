# 第 3 章 快速上手

这一章的目标是让你尽快把当前仓库跑起来，并且知道第一次启动时哪些步骤是必须的，哪些是可选的。

对当前 DeerFlow 来说，最常见的两条路径是：

- **Docker 开发模式**：最省心，适合快速体验，也适合大多数开发场景
- **本地开发模式**：适合你想改代码、看热重载、调试启动链路的时候

两条路径的共同前提只有两个：

1. 生成本地配置文件
2. 至少配置一个可用模型

## 3.1 生成配置文件

如果你是第一次进入仓库，先执行：

```bash
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow
make config
```

`make config` 会调用 `scripts/configure.py`，主要做三件事：

1. 生成根目录 `config.yaml`
2. 生成根目录 `.env`
3. 生成 `frontend/.env`

如果仓库里已经存在 `config.yaml`、`config.yml` 或 `configure.yml`，命令会直接中止，避免覆盖已有配置。

## 3.2 配置第一个模型

DeerFlow 启动前，至少需要一个可用模型。最简单的方式是在根目录 `config.yaml` 里配置一个 OpenAI 兼容模型。

例如：

```yaml
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY
    max_tokens: 4096
    temperature: 0.7
```

然后在根目录 `.env` 中写入：

```bash
OPENAI_API_KEY=your-openai-api-key
```

### 几个关键字段怎么理解

- `name`：模型在系统内部的标识
- `display_name`：前端展示名称
- `use`：模型类路径
- `model`：实际调用 provider 时使用的模型名
- `api_key: $OPENAI_API_KEY`：从环境变量取值

如果你用的是 OpenRouter 或其他 OpenAI 兼容网关，也可以继续用 `langchain_openai:ChatOpenAI`，只需要补上 `base_url`。

例如：

```yaml
models:
  - name: openrouter-gemini-2.5-flash
    display_name: Gemini 2.5 Flash (OpenRouter)
    use: langchain_openai:ChatOpenAI
    model: google/gemini-2.5-flash-preview
    api_key: $OPENAI_API_KEY
    base_url: https://openrouter.ai/api/v1
```

## 3.3 可选：配置搜索能力

如果你希望 agent 能联网搜索，通常还需要在 `.env` 中补充搜索相关密钥。

最常见的是：

```bash
TAVILY_API_KEY=your-tavily-api-key
INFOQUEST_API_KEY=your-infoquest-api-key
```

项目里的搜索与抓取能力来自工具配置和扩展模块，常见实现位于：

- `deerflow.community.tavily`
- `deerflow.community.infoquest`
- `deerflow.community.jina_ai`
- `deerflow.community.firecrawl`

如果只是先把系统跑起来，不一定要马上把这些都配齐；但做研究类任务时，缺少搜索工具会明显影响效果。

## 3.4 Sandbox 模式

Sandbox 决定了 agent 执行代码和操作文件的环境。

当前仓库最常见的三种使用方式可以这样理解：

### Local Sandbox

使用本地 provider，在宿主机上执行文件与命令相关操作。

典型类路径：

```yaml
sandbox:
  use: deerflow.sandbox.local:LocalSandboxProvider
```

这种模式更轻量，适合调试配置和本地开发，但隔离性弱。

### AIO Sandbox

使用容器化的一体化 sandbox provider。

典型类路径：

```yaml
sandbox:
  use: deerflow.community.aio_sandbox:AioSandboxProvider
```

这是当前最常见的容器化方式。AIO Sandbox 的代码在：

- `backend/packages/harness/deerflow/community/aio_sandbox/`

### Provisioner / Kubernetes 模式

仍然使用 `AioSandboxProvider`，但通过 `provisioner_url` 把 sandbox 调度到 Kubernetes 环境：

```yaml
sandbox:
  use: deerflow.community.aio_sandbox:AioSandboxProvider
  provisioner_url: http://provisioner:8002
```

这个模式更适合多人共享环境或生产场景。

## 3.5 Docker 开发模式

这是最推荐的起步方式。

### 第一步：准备 sandbox 镜像

```bash
make docker-init
```

这个命令会调用 `scripts/docker.sh init`。

它会做这些事：

- 读取 `config.yaml` 中的 sandbox 配置
- 判断当前是 `local`、`aio` 还是 `provisioner` 模式
- 在需要时预拉取 sandbox 镜像

如果你当前配置的是 local sandbox，它不会强制要求 Docker 镜像一定可用。

### 第二步：启动服务

```bash
make docker-start
```

启动后访问：

```text
http://localhost:2026
```

### Docker 模式下会启动哪些服务

默认情况下：

- `frontend`
- `gateway`
- `langgraph`
- `nginx`

如果检测到 `provisioner_url`，还会额外启动：

- `provisioner`

### Gateway mode

如果你想用 Gateway 内嵌 runtime 的模式：

```bash
make docker-start-pro
```

这时不会启动独立的 LangGraph 容器。

### 停止和看日志

```bash
make docker-stop
make docker-logs
make docker-logs-frontend
make docker-logs-gateway
```

## 3.6 本地开发模式

如果你需要看本地服务启动链路、改代码或调试热重载，用这条路径。

### 第一步：检查依赖

```bash
make check
```

当前检查脚本会验证：

- Node.js 22+
- pnpm
- uv
- nginx

### 第二步：安装依赖

```bash
make install
```

它会执行：

- `cd backend && uv sync`
- `cd frontend && pnpm install`

### 第三步：可选预拉取 sandbox 镜像

```bash
make setup-sandbox
```

如果你打算使用容器化 sandbox，建议先执行。

### 第四步：启动服务

```bash
make dev
```

当前本地启动链路由 `scripts/serve.sh` 管理。

默认会拉起：

- LangGraph Server：`localhost:2024`
- Gateway API：`localhost:8001`
- Frontend：`localhost:3000`
- Nginx：`localhost:2026`

你真正访问的地址仍然是：

```text
http://localhost:2026
```

### 本地 Gateway mode

如果你想直接体验 Gateway 内嵌 runtime：

```bash
make dev-pro
```

### 后台启动

如果你想把服务放到后台：

```bash
make dev-daemon
make dev-daemon-pro
```

### 停止服务

```bash
make stop
```

## 3.7 `make dev` 实际做了什么

理解这条命令很有帮助，因为它基本代表了 DeerFlow 本地开发时的真实启动流程。

执行 `make dev` 时，整体上会经历下面几步：

1. 运行 `scripts/check.py` 检查 Node.js、pnpm、uv、nginx
2. 运行 `scripts/serve.sh --dev`
3. 自动加载根目录 `.env`
4. 检查 `config.yaml` 是否存在
5. 自动执行 `scripts/config-upgrade.sh`
6. 同步 backend 与 frontend 依赖
7. 根据是否启用 Gateway mode，更新 `frontend/.env.local`
8. 启动 LangGraph、Gateway、Frontend、Nginx

因此，当前仓库的“开发入口”并不是某个单独服务，而是整个 `serve.sh` 编排脚本。

## 3.8 内嵌 Python Client

除了 Web UI，DeerFlow 还提供了 Python 内嵌客户端，代码位置是：

- `backend/packages/harness/deerflow/client.py`

使用时导入：

```python
from deerflow.client import DeerFlowClient
```

最简单的例子：

```python
from deerflow.client import DeerFlowClient

client = DeerFlowClient()
result = client.chat("帮我总结一下这个项目的架构", thread_id="demo-thread")
print(result)
```

它适合这些场景：

- 在 Python 脚本里直接调用 DeerFlow
- 写自动化测试
- 不启动完整 Web 服务时进行嵌入式集成

客户端还提供：

- 流式调用
- 模型列表查询
- skills 管理
- memory 访问
- uploads 管理
- MCP 配置访问

## 3.9 第一次启动时最容易卡住的地方

如果第一次运行失败，优先检查这几个点：

### 1. `config.yaml` 不存在

先执行：

```bash
make config
```

### 2. 模型没有配置或 API key 不可用

至少确认：

- `config.yaml` 中有 `models`
- `.env` 中有对应的 key

### 3. 本地缺少依赖

执行：

```bash
make check
```

如果缺：

- Node.js
- pnpm
- uv
- nginx

本地模式就跑不起来。

### 4. Docker 环境不可用

如果你走 Docker 模式但 Docker daemon 没启动，`docker-init` 或 `docker-start` 会失败。

### 5. sandbox 配置与运行方式不匹配

例如：

- 配了 AIO Sandbox，但本地没有 Docker
- 配了 provisioner，但没有 provisioner 环境

## 3.10 推荐的起步顺序

如果你现在的目标是“尽快跑起来并开始学习源码”，推荐顺序是：

1. `make config`
2. 在 `config.yaml` 配一个最简单可用的模型
3. 在 `.env` 写入 API key
4. 优先尝试 `make docker-init && make docker-start`
5. 如果你需要看本地启动链路，再用 `make check && make install && make dev`

## 小结

- **最小起步条件**：`config.yaml`、`.env`、至少一个可用模型
- **推荐入口**：Docker 模式用 `make docker-start`，本地模式用 `make dev`
- **核心启动脚本**：本地是 `scripts/serve.sh`，Docker 是 `scripts/docker.sh`
- **访问入口**：统一走 `http://localhost:2026`
- **嵌入式调用**：使用 `from deerflow.client import DeerFlowClient`
