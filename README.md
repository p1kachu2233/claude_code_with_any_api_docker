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

## 文件说明

* `claude-docker/Dockerfile`
  用于构建 Claude Code 运行环境

* `claude-docker/docker-compose.yml`
  用于启动 `claude-code` 和 `litellm`

* `claude-workspace/`
  工作目录，会挂载到容器里的 `/workspace`

* `litellm/config.yaml`
  用于配置 LiteLLM 的上游模型/provider

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

首次启动，或修改了 `Dockerfile` 之后：

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

保持一致

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

保持一致

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

## 更新项目

普通更新：

```bash
docker compose up -d
```

修改了 `Dockerfile` 后更新：

```bash
docker compose up -d --build
```

拉取远程镜像后更新：

```bash
docker compose pull
docker compose up -d
```

---

## 停止项目

```bash
docker compose down
```

> 不要随便执行 `docker compose down -v`，这会删除 volume。

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

停止：

```bash
docker compose down
```

---

## 说明

* `claude-workspace/` 是 Claude Code 的工作目录
* `litellm/config.yaml` 控制上游模型/provider
* 如果需要切换 provider，通常只需要修改：

  * `docker-compose.yml` 中的 API key 变量
  * `litellm/config.yaml` 中的 `model` 和 `api_key`
* 更新容器时，volume 通常会保留，除非显式删除
