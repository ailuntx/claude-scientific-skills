# 模态缩放与并发

## 目录

- [自动缩放](#自动缩放)
- [配置](#配置)
- [并行执行](#并行执行)
- [并发输入](#并发输入)
- [动态批处理](#动态批处理)
- [动态自动缩放器更新](#动态自动缩放器更新)
- [限制](#限制)

## 自动缩放

Modal 自动管理每个函数的容器池：
- 当新输入无可用容量时启动容器
- 关闭空闲容器以节省成本
- 支持从零（空闲时零成本）扩展到数千个容器

基础自动缩放无需配置——开箱即用。

## 配置

微调自动缩放行为：

```python
@app.function(
    max_containers=100,     # 容器数量上限
    min_containers=2,       # 保持2个预热容器（减少冷启动）
    buffer_containers=5,    # 预留5个额外容器应对突发流量
    scaledown_window=300,   # 关闭前等待5分钟空闲
)
def handle_request(data):
    ...
```

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `max_containers` | 无限制 | 容器总数硬性上限 |
| `min_containers` | 0 | 最小预热容器数（空闲时仍产生费用） |
| `buffer_containers` | 0 | 防止排队的额外容器 |
| `scaledown_window` | 60 | 关闭前的空闲等待秒数 |

### 权衡取舍

- 更高的 `min_containers` = 更低延迟，更高成本
- 更高的 `buffer_containers` = 更少排队，更高成本
- 更低的 `scaledown_window` = 更快节省成本，更多冷启动

## 并行执行

### `.map()` — 批量处理输入

```python
@app.function()
def process(item):
    return heavy_computation(item)

@app.local_entrypoint()
def main():
    items = list(range(10_000))
    results = list(process.map(items))
```

Modal 自动扩展容器处理工作负载，结果保持输入顺序。

### `.map()` 选项

```python
# 无序结果（更快）
for result in process.map(items, order_outputs=False):
    handle(result)

# 收集错误而非抛出异常
results = list(process.map(items, return_exceptions=True))
for r in results:
    if isinstance(r, Exception):
        print(f"错误: {r}")
```

### `.starmap()` — 多参数支持

```python
@app.function()
def add(x, y):
    return x + y

results = list(add.starmap([(1, 2), (3, 4), (5, 6)]))
# [3, 7, 11]
```

### `.spawn()` — 即发即弃

```python
# 立即返回
call = process.spawn(large_data)

# 稍后检查状态或获取结果
result = call.get()
```

最多支持100万个待处理的 `.spawn()` 调用。

## 并发输入

默认每个容器一次处理一个输入。使用 `@modal.concurrent` 实现多输入处理：

```python
@app.function(gpu="L40S")
@modal.concurrent(max_inputs=10)
async def predict(text: str):
    result = await model.predict_async(text)
    return result
```

此模式适用于I/O密集型工作负载或异步推理场景，单个GPU可同时处理多个请求。

### 结合Web端点

```python
@app.function(gpu="L40S")
@modal.concurrent(max_inputs=20)
@modal.asgi_app()
def web_service():
    return fastapi_app
```

## 动态批处理

将输入动态分组以提升GPU利用率：

```python
@app.function(gpu="L40S")
@modal.batched(max_batch_size=32, wait_ms=100)
async def batch_predict(texts: list[str]):
    # 单次最多处理32个文本
    embeddings = model.encode(texts)
    return list(embeddings)
```

- `max_batch_size` — 每批最大输入量
- `wait_ms` — 处理前等待更多输入的毫秒数
- 函数接收列表参数，必须返回等长结果列表

## 动态自动缩放器更新

无需重新部署即可运行时调整缩放策略：

```python
@app.function()
def scale_up_for_peak():
    process = modal.Function.from_name("my-app", "process")
    process.update_autoscaler(min_containers=10, buffer_containers=20)

@app.function()
def scale_down_after_peak():
    process = modal.Function.from_name("my-app", "process")
    process.update_autoscaler(min_containers=1, buffer_containers=2)
```

下次部署时设置将恢复为装饰器中的初始值。

## 限制

| 资源 | 限制 |
|----------|-------|
| 待处理输入（未分配） | 2,000 |
| 总输入（运行中+待处理） | 25,000 |
| 待处理 `.spawn()` 输入 | 1,000,000 |
| 单次 `.map()` 并发量 | 1,000 |
| 速率限制（Web端点） | 200 请求/秒 |

超出限制将触发 `Resource Exhausted` 错误，建议实现重试逻辑增强鲁棒性。
