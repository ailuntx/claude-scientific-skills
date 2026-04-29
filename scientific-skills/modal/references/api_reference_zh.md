# Modal API 参考

## 核心类

### modal.App

部署的基本单元。用于组织相关函数。

```python
app = modal.App("my-app")
```

| 方法 | 描述 |
|--------|-------------|
| `app.function(**kwargs)` | 注册函数的装饰器 |
| `app.cls(**kwargs)` | 注册类的装饰器 |
| `app.local_entrypoint()` | 本地入口点的装饰器 |

### modal.Function

由自动扩缩容器池支持的无服务器函数。

| 方法 | 描述 |
|--------|-------------|
| `.remote(*args)` | 在云端执行（同步） |
| `.local(*args)` | 本地执行 |
| `.spawn(*args)` | 异步执行，返回 `FunctionCall` |
| `.map(inputs)` | 对输入进行并行执行 |
| `.starmap(inputs)` | 多参数并行执行 |
| `.from_name(app, fn)` | 引用已部署函数 |
| `.update_autoscaler(**kwargs)` | 动态扩缩更新 |

### modal.Cls

具有生命周期钩子的无服务器类。

```python
@app.cls(gpu="L40S")
class MyClass:
    @modal.enter()
    def setup(self): ...  # 容器启动时执行

    @modal.method()
    def run(self, data): ...  # 暴露为可调用方法

    @modal.exit()
    def cleanup(self): ...  # 容器关闭时执行
```

| 装饰器 | 描述 |
|-----------|-------------|
| `@modal.enter()` | 容器启动钩子 |
| `@modal.exit()` | 容器关闭钩子 |
| `@modal.method()` | 暴露为可调用方法 |
| `@modal.parameter()` | 类级别参数 |

## 镜像

### modal.Image

定义容器环境。

| 方法 | 描述 |
|--------|-------------|
| `.debian_slim(python_version=)` | Debian 基础镜像 |
| `.from_registry(tag)` | Docker Hub 镜像 |
| `.from_dockerfile(path)` | 从 Dockerfile 构建 |
| `.micromamba(python_version=)` | Conda/mamba 基础 |
| `.uv_pip_install(*pkgs)` | 使用 uv 安装（推荐） |
| `.pip_install(*pkgs)` | 使用 pip 安装 |
| `.pip_install_from_requirements(path)` | 从文件安装 |
| `.apt_install(*pkgs)` | 安装系统包 |
| `.run_commands(*cmds)` | 执行 shell 命令 |
| `.run_function(fn)` | 构建期间运行 Python 函数 |
| `.add_local_dir(local, remote)` | 添加目录 |
| `.add_local_file(local, remote)` | 添加单个文件 |
| `.add_local_python_source(module)` | 添加 Python 模块 |
| `.env(dict)` | 设置环境变量 |
| `.imports()` | 远程导入的上下文管理器 |

## 存储

### modal.Volume

分布式持久化文件存储。

```python
vol = modal.Volume.from_name("name", create_if_missing=True)
```

| 方法 | 描述 |
|--------|-------------|
| `.from_name(name)` | 引用或创建存储卷 |
| `.commit()` | 强制立即提交 |
| `.reload()` | 刷新以查看其他容器的写入 |

挂载：`@app.function(volumes={"/path": vol})`

### modal.NetworkFileSystem

遗留共享存储（已被 Volume 取代）。

## 密钥

### modal.Secret

安全凭证注入。

| 方法 | 描述 |
|--------|-------------|
| `.from_name(name)` | 引用命名密钥 |
| `.from_dict(dict)` | 内联创建（仅开发） |
| `.from_dotenv()` | 从 .env 文件加载 |

用法：`@app.function(secrets=[modal.Secret.from_name("x")])`

函数内访问：`os.environ["KEY"]`

## 调度

### modal.Cron

```python
schedule = modal.Cron("0 9 * * *")  # Cron 语法
```

### modal.Period

```python
schedule = modal.Period(hours=6)  # 固定间隔
```

用法：`@app.function(schedule=modal.Cron("..."))`

## Web

### 装饰器

| 装饰器 | 描述 |
|-----------|-------------|
| `@modal.fastapi_endpoint()` | 简易 FastAPI 端点 |
| `@modal.asgi_app()` | 完整 ASGI 应用（FastAPI, Starlette） |
| `@modal.wsgi_app()` | 完整 WSGI 应用（Flask, Django） |
| `@modal.web_server(port=)` | 自定义 Web 服务器 |

### 函数修饰符

| 装饰器 | 描述 |
|-----------|-------------|
| `@modal.concurrent(max_inputs=)` | 单容器处理多输入 |
| `@modal.batched(max_batch_size=, wait_ms=)` | 动态输入批处理 |

## GPU 字符串

| 字符串 | GPU |
|--------|-----|
| `"T4"` | NVIDIA T4 16GB |
| `"L4"` | NVIDIA L4 24GB |
| `"A10"` | NVIDIA A10 24GB |
| `"L40S"` | NVIDIA L40S 48GB |
| `"A100-40GB"` | NVIDIA A100 40GB |
| `"A100-80GB"` | NVIDIA A100 80GB |
| `"H100"` | NVIDIA H100 80GB |
| `"H100!"` | H100（禁止自动升级） |
| `"H200"` | NVIDIA H200 141GB |
| `"B200"` | NVIDIA B200 192GB |
| `"B200+"` | B200 或 B300，按 B200 计费 |
| `"H100:4"` | 4x H100 |

## CLI 命令

| 命令 | 描述 |
|---------|-------------|
| `modal setup` | 身份验证 |
| `modal run <file>` | 运行本地入口点 |
| `modal serve <file>` | 带热重载的开发服务器 |
| `modal deploy <file>` | 生产环境部署 |
| `modal app list` | 列出已部署应用 |
| `modal app stop <name>` | 停止应用 |
| `modal volume create <name>` | 创建存储卷 |
| `modal volume ls <name>` | 列出存储卷文件 |
| `modal volume put <name> <file>` | 上传至存储卷 |
| `modal volume get <name> <file>` | 从存储卷下载 |
| `modal secret create <name> K=V` | 创建密钥 |
| `modal secret list` | 列出密钥 |
| `modal secret delete <name>` | 删除密钥 |
| `modal token set` | 设置认证令牌 |
