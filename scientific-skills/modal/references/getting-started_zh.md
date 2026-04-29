# Modal 入门指南

## 安装

使用 uv（推荐）或 pip 安装 Modal：

```bash
# 推荐方式
uv pip install modal

# 替代方式
pip install modal
```

## 认证

### 交互式设置

```bash
modal setup
```

此命令将打开浏览器进行认证，并在本地存储凭证。

### 无界面 / CI/CD 设置

在无浏览器环境中，使用基于令牌的认证：

1. 在 https://modal.com/settings 生成令牌
2. 设置环境变量：

```bash
export MODAL_TOKEN_ID=<your-token-id>
export MODAL_TOKEN_SECRET=<your-token-secret>
```

或使用 CLI：

```bash
modal token set --token-id <id> --token-secret <secret>
```

### 免费额度

Modal 每月提供 30 美元免费额度。免费额度无需信用卡。

## 你的第一个应用

### Hello World

创建文件 `hello.py`：

```python
import modal

app = modal.App("hello-world")

@app.function()
def greet(name: str) -> str:
    return f"Hello, {name}! This ran in the cloud."

@app.local_entrypoint()
def main():
    result = greet.remote("World")
    print(result)
```

运行：

```bash
modal run hello.py
```

执行过程：
1. Modal 打包你的代码
2. 在云端创建容器
3. 远程执行 `greet()`
4. 将结果返回到本地机器

### 理解执行流程

- `modal.App("name")` — 创建命名应用
- `@app.function()` — 标记函数用于远程执行
- `@app.local_entrypoint()` — 定义本地入口点（在本地机器运行）
- `.remote()` — 在云端调用函数
- `.local()` — 本地调用函数（用于测试）

### 运行模式

| 命令 | 描述 |
|---------|-------------|
| `modal run script.py` | 运行 `@app.local_entrypoint()` 函数 |
| `modal serve script.py` | 启动带热重载的开发服务器（用于 Web 端点） |
| `modal deploy script.py` | 部署到生产环境（持久化） |

### 简易网页爬虫

```python
import modal

app = modal.App("web-scraper")

image = modal.Image.debian_slim().uv_pip_install("httpx", "beautifulsoup4")

@app.function(image=image)
def scrape(url: str) -> str:
    import httpx
    from bs4 import BeautifulSoup

    response = httpx.get(url)
    soup = BeautifulSoup(response.text, "html.parser")
    return soup.get_text()[:1000]

@app.local_entrypoint()
def main():
    result = scrape.remote("https://example.com")
    print(result)
```

### GPU 加速推理

```python
import modal

app = modal.App("gpu-inference")

image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install("torch", "transformers", "accelerate")
)

@app.function(gpu="L40S", image=image)
def generate(prompt: str) -> str:
    from transformers import pipeline
    pipe = pipeline("text-generation", model="gpt2", device="cuda")
    result = pipe(prompt, max_length=100)
    return result[0]["generated_text"]

@app.local_entrypoint()
def main():
    print(generate.remote("The future of AI is"))
```

## 项目结构

Modal 应用通常是单个 Python 文件，也可组织为模块：

```
my-project/
├── app.py           # 主应用（含 @app.local_entrypoint()）
├── inference.py     # 推理函数
├── training.py      # 训练函数
└── common.py        # 共享工具
```

使用 `modal.Image.add_local_python_source()` 将本地模块包含到容器镜像中。

## 核心概念摘要

| 概念 | 功能 |
|---------|-------------|
| `App` | 将相关函数分组为可部署单元 |
| `Function` | 由自动伸缩容器支持的无服务器函数 |
| `Image` | 定义容器环境（软件包、文件） |
| `Volume` | 持久化分布式文件存储 |
| `Secret` | 安全凭证注入 |
| `Schedule` | 定时或周期性任务调度 |
| `gpu` | 函数的 GPU 类型/数量配置 |

## 后续步骤

- 查看 `functions.md` 了解高级函数模式
- 查看 `images.md` 了解自定义容器环境
- 查看 `gpu.md` 了解 GPU 选择与配置
- 查看 `web-endpoints.md` 了解 API 服务搭建
