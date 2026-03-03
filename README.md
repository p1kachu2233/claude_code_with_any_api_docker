# Claude Code Docker Setup

一个最小可用的 Docker 项目，用于在容器中运行 Claude Code，并通过 LiteLLM 连接上游模型。

## 目录结构

```text
.
├─ claude-docker/
│  ├─ Dockerfile
│  └─ docker-compose.yml
├─ claude-workspace/
│  └─ helloword.txt
└─ litellm/
   └─ config.yaml
```

## 项目说明

本项目包含两个服务：

* **litellm**：作为 Claude Code 的上游代理，负责转发请求到实际 provider
* **claude-code**：Claude Code 运行容器，用于在 `/workspace` 中读取和修改文件

其中：

* 本机 `claude-workspace/` 会挂载到容器内的 `/workspace`
* `/home/claude` 使用 Docker volume 持久化，用于保存 Claude 用户目录、配置、缓存，以及当前安装的 Claude Code

---

## 文件说明

### `claude-docker/Dockerfile`

用于构建 Claude Code 运行环境。

### `claude-docker/docker-compose.yml`

用于启动：

* `claude-code`
* `litellm`

### `claude-workspace/`

工作目录，会挂载到容器里的：

```text
/workspace
```

### `litellm/config.yaml`

用于配置 LiteLLM 的上游模型和认证信息。

---

## 获取项目

```bash
git clone <仓库地址>
cd <仓库目录>
```

例如：

```bash
git clone https://github.com/<用户名>/<仓库名>.git
cd <仓库名>
```

---

## 启动项目

进入 compose 所在目录：

```bash
cd claude-docker
```

首次启动，或修改了 `Dockerfile` / `docker-compose.yml` 之后：

```bash
docker compose up -d --build
```

普通启动：

```bash
docker compose up -d
```

查看状态：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

---

## 进入 Claude Code

先进入 Claude 容器：

```bash
docker exec -it claude-code bash
```

然后进入工作目录并启动 Claude Code：

```bash
cd /workspace
claude
```

---

## 工作目录说明

本机目录：

```text
claude-workspace/
```

会挂载到容器内：

```text
/workspace
```

也就是说，放到 `claude-workspace/` 里的文件，都可以在 Claude Code 中直接读取和修改。

---

## `docker-compose.yml` 中可修改的内容

### 1. 工作目录挂载路径

例如：

```yaml
- C:\claude-code\claude-workspace:/workspace
```

说明：

* 左边 `C:\claude-code\claude-workspace` 是本机路径
* 右边 `/workspace` 是容器路径

如果项目不在 `C:` 盘，可以改成自己的真实路径，例如：

```yaml
- D:\my-project\claude-workspace:/workspace
```

---

### 2. LiteLLM 配置文件路径

例如：

```yaml
- C:\claude-code\litellm\config.yaml:/app/config.yaml
```

说明：

* 左边是本机 `config.yaml` 路径
* 右边是容器内 LiteLLM 读取配置的位置

如果项目路径变化，左边也要一起改。

---

### 3. LiteLLM 暴露端口

例如：

```yaml
ports:
  - "4000:4000"
```

说明：

* 左边 `4000` 是本机端口
* 右边 `4000` 是容器端口

如果本机 `4000` 被占用，可以改成：

```yaml
ports:
  - "4001:4000"
```

如果改了这里，`ANTHROPIC_BASE_URL` 也要一起改。

---

### 4. `ANTHROPIC_BASE_URL`

例如：

```yaml
- ANTHROPIC_BASE_URL=http://host.docker.internal:4000
```

说明：

* 这是 Claude Code 连接 LiteLLM 的地址
* `4000` 要和 LiteLLM 对外暴露的本机端口一致

如果端口改成了 `4001`，这里也要改成：

```yaml
- ANTHROPIC_BASE_URL=http://host.docker.internal:4001
```

---

### 5. `ANTHROPIC_AUTH_TOKEN`

例如：

```yaml
- ANTHROPIC_AUTH_TOKEN=sk-claudecode-local
```

说明：

* 这是 Claude Code 访问 LiteLLM 时使用的认证 token
* 它必须和 `litellm/config.yaml` 里的 `master_key` 保持一致

---

### 6. `ANTHROPIC_MODEL`

例如：

```yaml
- ANTHROPIC_MODEL=cerebras-gpt-oss
```

说明：

* 这个值不是上游真实模型名
* 它是 LiteLLM 中定义的 `model_name`
* 需要和 `config.yaml` 中的 `model_name` 一致

---

### 7. API Key 环境变量

例如：

```yaml
environment:
  - CEREBRAS_API_KEY=your_api_key_here
```

说明：

* 这是提供给 LiteLLM 读取的上游 provider API key
* 如果换 provider，这里的变量名通常也要一起改，例如：

  * `OPENAI_API_KEY`
  * `ANTHROPIC_API_KEY`
  * `DEEPSEEK_API_KEY`

---

### 8. `claude_user_home` volume

例如：

```yaml
volumes:
  - claude_user_home:/home/claude
```

说明：

* 该 volume 会持久化整个 `/home/claude`
* Claude Code 的用户级安装、配置、缓存、登录状态通常都会保存在这里
* **当前实际运行的 Claude 版本，通常由这个 volume 里的内容决定**

---

## `litellm/config.yaml` 中可修改的内容

示例：

```yaml
model_list:
  - model_name: cerebras-gpt-oss
    litellm_params:
      model: cerebras/gpt-oss-120b
      api_key: os.environ/CEREBRAS_API_KEY
      temperature: 0.2
      top_p: 0.9
      max_tokens: 4096

general_settings:
  master_key: sk-claudecode-local
```

### 1. `model_name`

例如：

```yaml
model_name: cerebras-gpt-oss
```

说明：

* 这是给 Claude Code 使用的模型别名
* 需要和 `docker-compose.yml` 中的：

```yaml
ANTHROPIC_MODEL=cerebras-gpt-oss
```

保持一致。

---

### 2. `model`

例如：

```yaml
model: cerebras/gpt-oss-120b
```

说明：

* 这是 LiteLLM 实际调用的上游模型名
* 如果换 provider，这里改成对应格式即可，例如：

  * `openai/gpt-4o-mini`
  * `anthropic/claude-3-5-sonnet`
  * `deepseek/deepseek-chat`

---

### 3. `api_key`

例如：

```yaml
api_key: os.environ/CEREBRAS_API_KEY
```

说明：

* 表示从环境变量中读取 API key
* 如果换 provider，通常这里也要换成对应变量名，例如：

```yaml
api_key: os.environ/OPENAI_API_KEY
```

---

### 4. 推理参数

例如：

```yaml
temperature: 0.2
top_p: 0.9
max_tokens: 4096
```

说明：

* `temperature`：越低越稳定
* `top_p`：控制采样范围
* `max_tokens`：最大输出长度

这些都可以按需修改。

---

### 5. `master_key`

例如：

```yaml
master_key: sk-claudecode-local
```

说明：

* 这是 LiteLLM 自身的认证 key
* 需要和 `docker-compose.yml` 中的：

```yaml
ANTHROPIC_AUTH_TOKEN=sk-claudecode-local
```

保持一致。

---

## 使用流程

1. 把要处理的文件放进：

```text
claude-workspace/
```

2. 启动项目：

```bash
cd claude-docker
docker compose up -d
```

3. 进入 Claude 容器：

```bash
docker exec -it claude-code bash
```

4. 在容器中进入工作目录并启动 Claude：

```bash
cd /workspace
claude
```

---

## 更新说明

### 先理解一件事

当前 Dockerfile 中，Claude Code 是安装在：

```text
/home/claude
```

而 `docker-compose.yml` 又把整个 `/home/claude` 持久化到了 volume：

```yaml
- claude_user_home:/home/claude
```

这意味着：

* **镜像里安装的 Claude，只会在 volume 初次创建时复制进去**
* 一旦 volume 已存在，后续容器启动时 `/home/claude` 会被 volume 覆盖
* 因此：

```bash
docker compose up -d --build
```

**不能保证更新当前实际使用的 Claude Code 版本**

它能更新的是：

* Dockerfile
* 容器镜像
* 系统依赖
* Compose 配置

但**不一定会更新 `/home/claude` volume 里已经存在的 Claude Code**

---

## 推荐更新方式

### 方案一：保留现有 volume，手动升级 Claude Code

适合：

* 保留当前配置、缓存、登录状态
* 不想删除 `/home/claude`
* 更关注方便

进入容器执行：

```bash
docker exec -it claude-code bash
curl -fsSL https://claude.ai/install.sh | bash
```

或者一条命令执行：

```bash
docker exec -it claude-code bash -lc 'curl -fsSL https://claude.ai/install.sh | bash'
```

升级后检查版本：

```bash
docker exec -it claude-code bash -lc 'which claude && claude --version'
```

说明：

* 这种方式会直接更新 volume 中的 Claude Code
* 容器重启后版本仍会保留
* **这是当前项目结构下最实用的 Claude 更新方式**

---

### 方案二：删除 `claude_user_home` volume 后重建

适合：

* 想要完全干净地重装 Claude Code
* 接受丢失 `/home/claude` 中的配置、缓存、登录状态

停止容器：

```bash
docker compose down
```

查看 volume：

```bash
docker volume ls
```

删除对应 volume：

```bash
docker volume rm <项目名>_claude_user_home
```

重新构建并启动：

```bash
docker compose up -d --build
```

说明：

* 删除 volume 后，新的 `/home/claude` 会重新从镜像初始化
* 这种方式能确保镜像中的 Claude 安装重新生效
* 代价是会丢失原有用户目录数据

---

### 方案三：只更新镜像和容器环境

适合：

* 修改了 `Dockerfile`
* 修改了 compose 配置
* 更新系统依赖
* 但**不依赖它更新 Claude Code 本体**

执行：

```bash
docker compose up -d --build
```

说明：

* 这会更新镜像和容器
* **不应被视为 Claude Code 的可靠升级方式**
* 如果你要升级的是 Claude Code 本身，请优先使用“方案一”或“方案二”

---

## 更新建议

### 日常使用建议

如果只是平时使用，推荐：

1. 保留 `claude_user_home` volume
2. 需要升级 Claude Code 时，在容器中手动执行安装脚本
3. 需要更新系统环境时，再执行：

```bash
docker compose up -d --build
```

也就是说：

* **Claude Code 本体更新**：手动执行安装脚本
* **镜像 / 环境更新**：`docker compose up -d --build`

这两者不是一回事。

---

## 停止项目

```bash
docker compose down
```

> 不要随便执行 `docker compose down -v`，这会删除 volume，包括 `claude_user_home`。

---

## 常用命令

启动：

```bash
docker compose up -d
```

构建并启动：

```bash
docker compose up -d --build
```

查看状态：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f
```

进入 Claude 容器：

```bash
docker exec -it claude-code bash
```

手动升级 Claude Code：

```bash
docker exec -it claude-code bash -lc 'curl -fsSL https://claude.ai/install.sh | bash'
```

查看 Claude 版本：

```bash
docker exec -it claude-code bash -lc 'which claude && claude --version'
```

停止：

```bash
docker compose down
```

---

## 说明

* `claude-workspace/` 是 Claude Code 的工作目录

* `litellm/config.yaml` 控制上游模型/provider

* 如果需要切换 provider，通常只需要修改：

  * `docker-compose.yml` 中的 API key 环境变量
  * `litellm/config.yaml` 中的 `model` 和 `api_key`

* 当前项目结构下，`/home/claude` 是持久化目录，因此：

  * Claude 配置会保留
  * Claude 缓存会保留
  * Claude 登录状态通常会保留
  * Claude Code 的用户级安装通常也会保留

---
