# Modal 函数与类

## 目录

- [函数](#函数)
- [远程执行](#远程执行)
- [带生命周期钩子的类](#带生命周期钩子的类)
- [并行执行](#并行执行)
- [异步函数](#异步函数)
- [本地入口点](#本地入口点)
- [生成器](#生成器)

## 函数

### 基础函数

```python
import modal

app = modal.App("my-app")

@app.function()
def compute(x: int, y: int) -> int:
    return x + y
```

### 函数参数

`@app.function()` 装饰器接受：

| 参数 | 类型 | 描述 |
|-----------|------|-------------|
| `image` | `Image` | 容器镜像 |
| `gpu` | `str` | GPU 类型（如 `"H100"`, `"A100:2"`） |
| `cpu` | `float` | CPU 核心数 |
| `memory` | `int` | 内存（单位 MiB） |
| `timeout` | `int` | 最大执行时间（秒） |
| `secrets` | `list[Secret]` | 注入的密钥 |
| `volumes` | `dict[str, Volume]` | 挂载的卷 |
| `schedule` | `Schedule` | 定时或周期调度 |
| `max_containers` | `int` | 最大容器数量 |
| `min_containers` | `int` | 最小预热容器数 |
| `retries` | `int` | 失败重试次数 |
| `concurrency_limit` | `int` | 最大并发输入数 |
| `ephemeral_disk` | `int` | 磁盘空间（单位 MiB） |

## 远程执行

### `.remote()` — 同步调用

```python
result = compute.remote(3, 4)  # 在云端运行，阻塞直至完成
```

### `.local()` — 本地执行

```python
result = compute.local(3, 4)  # 本地运行（用于测试）
```

### `.spawn()` — 异步触发后不管

```python
call = compute.spawn(3, 4)  # 立即返回
# ... 执行其他任务 ...
result = call.get()  # 稍后获取结果
```

`.spawn()` 支持最多 100 万个待处理输入。

## 带生命周期钩子的类

使用 `@app.cls()` 处理需要一次性加载资源的有状态工作负载：

```python
@app.cls(gpu="L40S", image=image)
class Model:
    @modal.enter()
    def setup(self):
        """在容器启动时运行一次"""
        import torch
        self.model = torch.load("/weights/model.pt")
        self.model.eval()

    @modal.method()
    def predict(self, text: str) -> dict:
        """可远程调用的方法"""
        return self.model(text)

    @modal.exit()
    def teardown(self):
        """容器关闭时运行"""
        cleanup_resources()
```

### 生命周期装饰器

| 装饰器 | 执行时机 |
|-----------|-------------|
| `@modal.enter()` | 容器启动时运行一次，在任何输入前 |
| `@modal.method()` | 每次远程调用时运行 |
| `@modal.exit()` | 容器关闭时运行 |

### 调用类方法

```python
# 创建实例并调用方法
model = Model()
result = model.predict.remote("Hello world")

# 并行调用
results = list(model.predict.map(["text1", "text2", "text3"]))
```

### 参数化类

```python
@app.cls()
class Worker:
    model_name: str = modal.parameter()

    @modal.enter()
    def load(self):
        self.model = load_model(self.model_name)

    @modal.method()
    def run(self, data):
        return self.model(data)

# 不同模型实例自动独立扩缩容
gpt = Worker(model_name="gpt-4")
llama = Worker(model_name="llama-3")
```

## 并行执行

### `.map()` — 并行处理

跨容器处理多个输入：

```python
@app.function()
def process(item):
    return heavy_computation(item)

@app.local_entrypoint()
def main():
    items = list(range(1000))
    results = list(process.map(items))
    print(f"已处理 {len(results)} 个项目")
```

- 结果按输入顺序返回
- Modal 自动扩缩容器处理工作负载
- 使用 `return_exceptions=True` 收集错误而非抛出异常

### `.starmap()` — 多参数并行

```python
@app.function()
def add(x, y):
    return x + y

results = list(add.starmap([(1, 2), (3, 4), (5, 6)]))
# [3, 7, 11]
```

### `.map()` 配合 `order_outputs=False`

当顺序无关时提升吞吐量：

```python
for result in process.map(items, order_outputs=False):
    handle(result)  # 结果按完成顺序到达
```

## 异步函数

Modal 原生支持 async/await：

```python
@app.function()
async def fetch_data(url: str) -> str:
    import httpx
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text
```

异步函数特别适合配合 `@modal.concurrent()` 处理每个容器的多个请求。

## 本地入口点

`@app.local_entrypoint()` 在本地运行并编排远程调用：

```python
@app.local_entrypoint()
def main():
    # 此代码在本地运行
    data = load_local_data()

    # 这些调用在云端运行
    results = list(process.map(data))

    # 返回本地处理
    save_results(results)
```

可定义多个入口点并按函数名选择：

```bash
modal run script.py::train
modal run script.py::evaluate
```

## 生成器

函数可实时生成结果：

```python
@app.function()
def generate_data():
    for i in range(100):
        yield process(i)

@app.local_entrypoint()
def main():
    for result in generate_data.remote_gen():
        print(result)
```

## 重试机制

配置失败自动重试：

```python
@app.function(retries=3)
def flaky_operation():
    ...
```

精细控制使用 `modal.Retries`：

```python
@app.function(retries=modal.Retries(max_retries=3, backoff_coefficient=2.0))
def api_call():
    ...
```

## 超时设置

设置最大执行时间：

```python
@app.function(timeout=3600)  # 1小时
def long_training():
    ...
```

默认超时为 300 秒（5 分钟），最大为 86400 秒（24 小时）。
