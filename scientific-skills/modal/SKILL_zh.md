---
name: modal
description: 在GPU和无服务器基础设施上运行Python的云计算平台。适用于部署AI/ML模型、运行GPU加速工作负载、提供Web端点、调度批处理作业或将Python代码扩展至云端。当用户提及Modal、无服务器GPU计算、将ML模型部署到云端、提供推理端点、在云端运行批处理或需要将Python工作负载扩展到本地机器之外时使用此技能。同样适用于用户希望在H100、A100或其他云端GPU上运行代码，或需要为模型创建Web API的场景。
license: Apache-2.0
metadata:
  skill-author: K-Dense Inc.
---

# Modal

## 概述

Modal是一个用于无服务器运行Python代码的云平台，专注于AI/ML工作负载。核心能力包括：
- **按需GPU计算**（支持T4、L4、A10、L40S、A100、H100、H200、B200）
- **无服务器函数**，支持从零到数千个容器的自动扩展
- **完全通过Python代码构建的自定义容器镜像**
- 通过Volume实现**持久化存储**，用于存储模型权重和数据集
- 用于托管模型和API的**Web端点**
- 通过cron或固定间隔调度的**定时任务**
- **亚秒级冷启动**，实现低延迟推理

Modal中的所有内容都通过代码定义——无需YAML或Dockerfile（但两者均受支持）。

## 何时使用此技能

在以下场景使用此技能：
- 在云端部署或托管AI/ML模型
- 运行GPU加速计算（训练、推理、微调）
- 创建无服务器Web API或端点
- 并行扩展批处理作业
- 调度周期性任务（数据管道、模型重训、网络爬取）
- 需要持久化云存储来保存模型权重或数据集
- 希望在自定义容器环境中运行代码
- 构建任务队列或异步任务处理系统

## 安装与认证

### 安装

```bash
uv pip install modal
```

### 认证

优先使用现有凭证，再创建新凭证：

1. 检查当前环境中是否已存在`MODAL_TOKEN_ID`和`MODAL_TOKEN_SECRET`。
2. 若不存在，检查本地`.env`文件中是否有这些值，并根据工作流需要加载。
3. 仅当上述来源均未提供凭证时，才通过交互式`modal setup`或生成新令牌。

```bash
modal setup
```

此命令将打开浏览器进行认证。对于CI/CD或无头环境，使用环境变量：

```bash
export MODAL_TOKEN_ID=<your-token-id>
export MODAL_TOKEN_SECRET=<your-token-secret>
```

若环境或`.env`中无可用令牌，请通过 https://modal.com/settings 生成。

Modal提供每月$30额度的免费层级。

**参考**：详细设置和首个应用教程见`references/getting-started.md`。

## 核心概念

### 应用与函数

Modal的`App`用于分组相关函数。被`@app.function()`装饰的函数将在云端远程运行：

```python
import modal

app = modal.App("my-app")

@app.function()
def square(x):
    return x ** 2

@app.local_entrypoint()
def main():
    # .remote()在云端运行
    print(square.remote(42))
```

通过`modal run script.py`运行，通过`modal deploy script.py`部署。

**参考**：生命周期钩子、类、`.map()`、`.spawn()`等详见`references/functions.md`。

### 容器镜像

Modal通过Python代码构建容器镜像。推荐使用`uv`作为包安装器：

```python
image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install("torch==2.8.0", "transformers", "accelerate")
    .apt_install("git")
)

@app.function(image=image)
def inference(prompt):
    from transformers import pipeline
    pipe = pipeline("text-generation", model="meta-llama/Llama-3-8B")
    return pipe(prompt)
```

关键镜像方法：
- `.uv_pip_install()` — 使用uv安装Python包（推荐）
- `.pip_install()` — 使用pip安装（备用）
- `.apt_install()` — 安装系统包
- `.run_commands()` — 构建期间执行shell命令
- `.run_function()` — 构建期间运行Python代码（如下载模型权重）
- `.add_local_python_source()` — 添加本地模块
- `.env()` — 设置环境变量

**参考**：Dockerfile、micromamba、缓存、GPU构建步骤详见`references/images.md`。

### GPU计算

通过`gpu`参数请求GPU：

```python
@app.function(gpu="H100")
def train_model():
    import torch
    device = torch.device("cuda")
    # GPU训练代码

# 多GPU
@app.function(gpu="H100:4")
def distributed_training():
    ...

# GPU回退链
@app.function(gpu=["H100", "A100-80GB", "A100-40GB"])
def flexible_inference():
    ...
```

可用GPU：T4、L4、A10、L40S、A100-40GB、A100-80GB、H100、H200、B200、B200+

- 单容器最多支持8个GPU（A10除外：最多4个）
- L40S推荐用于推理（性价比高，48GB显存）
- H100/A100可免费自动升级至H200/A100-80GB
- 使用`gpu="H100!"`可禁用自动升级

**参考**：GPU选择指南和多GPU训练详见`references/gpu.md`。

### Volume（持久化存储）

Volume提供分布式持久化文件存储：

```python
vol = modal.Volume.from_name("model-weights", create_if_missing=True)

@app.function(volumes={"/data": vol})
def save_model():
    # 写入挂载路径
    with open("/data/model.pt", "wb") as f:
        torch.save(model.state_dict(), f)

@app.function(volumes={"/data": vol})
def load_model():
    model.load_state_dict(torch.load("/data/model.pt"))
```

- 针对写一次读多次场景优化（模型权重、数据集）
- CLI操作：`modal volume ls`、`modal volume put`、`modal volume get`
- 后台每数秒自动提交

**参考**：v2卷、并发写入和最佳实践详见`references/volumes.md`。

### 密钥

安全地向函数传递凭证：

```python
@app.function(secrets=[modal.Secret.from_name("my-api-keys")])
def call_api():
    import os
    api_key = os.environ["API_KEY"]
    # 使用密钥
```

通过CLI创建密钥：`modal secret create my-api-keys API_KEY=sk-xxx`

或从`.env`文件创建：`modal.Secret.from_dotenv()`

**参考**：仪表盘设置、多密钥和模板详见`references/secrets.md`。

### Web端点

将模型和API托管为Web端点：

```python
@app.function()
@modal.fastapi_endpoint()
def predict(text: str):
    return {"result": model.predict(text)}
```

- `modal serve script.py` — 开发模式（热重载+临时URL）
- `modal deploy script.py` — 生产部署（永久URL）
- 支持FastAPI、ASGI（Starlette、FastHTML）、WSGI（Flask、Django）、WebSockets
- 请求体最大4GiB，响应大小无限制

**参考**：ASGI/WSGI应用、流式传输、认证和WebSockets详见`references/web-endpoints.md`。

### 定时任务

按计划运行函数：

```python
@app.function(schedule=modal.Cron("0 9 * * *"))  # UTC时间每日9点
def daily_pipeline():
    # ETL/重训/爬取等
    ...

@app.function(schedule=modal.Period(hours=6))
def periodic_check():
    ...
```

通过`modal deploy script.py`部署以激活计划。

- `modal.Cron("...")` — 标准cron语法，部署间保持稳定
- `modal.Period(hours=N)` — 固定间隔，重新部署时重置
- 在Modal仪表盘中监控运行状态

**参考**：cron语法和管理详见`references/scheduled-jobs.md`。

### 扩展与并发

Modal自动扩展容器。可配置限制：

```python
@app.function(
    max_containers=100,    # 上限
    min_containers=2,      # 保持预热以降低延迟
    buffer_containers=5,   # 预留容量
    scaledown_window=300,  # 空闲关闭等待时间（秒）
)
def process(data):
    ...
```

通过`.map()`并行处理输入：

```python
results = list(process.map([item1, item2, item3, ...]))
```

启用单容器并发请求处理：

```python
@app.function()
@modal.concurrent(max_inputs=10)
async def handle_request(req):
    ...
```

**参考**：`.map()`、`.starmap()`、`.spawn()`和限制详见`references/scaling.md`。

### 资源配置

```python
@app.function(
    cpu=4.0,              # 物理核心数（非vCPU）
    memory=16384,         # MiB
    ephemeral_disk=51200, # MiB（最高3TiB）
    timeout=3600,         # 秒
)
def heavy_computation():
    ...
```

默认值：0.125 CPU核心，128 MiB内存。按max(请求量, 实际使用量)计费。

**参考**：限制和计费细节详见`references/resources.md`。

## 含生命周期钩子的类

适用于有状态工作负载（如加载一次模型后处理多个请求）：

```python
@app.cls(gpu="L40S", image=image)
class Predictor:
    @modal.enter()
    def load_model(self):
        self.model = load_heavy_model()  # 容器启动时运行一次

    @modal.method()
    def predict(self, text: str):
        return self.model(text)

    @modal.exit()
    def cleanup(self):
        ...  # 容器关闭时运行
```

调用方式：`Predictor().predict.remote("hello")`

## 常见工作流模式

### GPU模型推理服务

```python
import modal

app = modal.App("llm-service")

image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install("vllm")
)

@app.cls(gpu="H100", image=image, min_containers=1)
class LLMService:
    @modal.enter()
    def load(self):
        from vllm import LLM
        self.llm = LLM(model="meta-llama/Llama-3-70B")

    @modal.method()
    @modal.fastapi_endpoint(method="POST")
    def generate(self, prompt: str, max_tokens: int = 256):
        outputs = self.llm.generate([prompt], max_tokens=max_tokens)
        return {"text": outputs[0].outputs[0].text}
```

### 批处理流水线

```python
app = modal.App("batch-pipeline")
vol = modal.Volume.from_name("pipeline-data", create_if_missing=True)

@app.function(volumes={"/data": vol}, cpu=4.0, memory=8192)
def process_chunk(chunk_id: int):
    import pandas as pd
    df = pd.read_parquet(f"/data/input/chunk_{chunk_id}.parquet")
    result = heavy_transform(df)
    result.to_parquet(f"/data/output/chunk_{chunk_id}.parquet")
    return len(result)

@app.local_entrypoint()
def main():
    chunk_ids = list(range(100))
    results = list(process_chunk.map(chunk_ids))
    print(f"共处理 {sum(results)} 行数据")
```

### 定时数据流水线

```python
app = modal.App("etl-pipeline")

@app.function(
    schedule=modal.Cron("0 */6 * * *"),  # 每6小时
    secrets=[modal.Secret.from_name("db-credentials")],
)
def etl_job():
    import os
    db_url = os.environ["DATABASE_URL"]
    # 抽取/转换/加载
    ...
```

## CLI参考

| 命令 | 描述 |
|---------|-------------|
| `modal setup` | Modal认证 |
| `modal run script.py` | 运行脚本的本地入口点 |
| `modal serve script.py` | 带热重载的开发服务器 |
| `modal deploy script.py` | 部署到生产环境 |
| `modal volume ls <name>` | 列出卷内文件 |
| `modal volume put <name> <file>` | 上传文件到卷 |
| `modal volume get <name> <file>` | 从卷下载文件 |
| `modal secret create <name> K=V` | 创建密钥 |
| `modal secret list` | 列出密钥 |
| `modal app list` | 列出已部署应用 |
| `modal app stop <name>` | 停止已部署应用 |

## 参考文件

各主题详细文档：

- `references/getting-started.md` — 安装、认证、首个应用
- `references/functions.md` — 函数、类、生命周期钩子、远程执行
- `references/images.md` — 容器镜像、包安装、缓存
- `references/gpu.md` — GPU类型、选择、多GPU、训练
- `references/volumes.md` — 持久化存储、文件管理、v2卷
- `references/secrets.md` — 凭证、环境变量、dotenv
- `references/web-endpoints.md` — FastAPI、ASGI/WSGI、流式传输、认证、WebSockets
- `references/scheduled-jobs.md` — Cron、周期计划、管理
- `references/scaling.md` — 自动扩展、并发、.map()、限制
- `references/resources.md` — CPU、内存、磁盘、超时配置
- `references/examples.md` — 常见用例和模式
- `references/api_reference.md` — 关键API类和方法

当需要超出本概述的详细信息时，请查阅这些文件。
