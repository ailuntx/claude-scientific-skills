# Forge API 参考文档

## 概述

Forge 是 EvolutionaryScale 提供的可扩展蛋白质设计与推理云平台。该平台提供对完整 ESM3 模型系列的 API 访问权限，包括无法在本地运行的大型模型。

**核心优势：**
- 支持访问所有 ESM3 模型（含 980 亿参数版本）
- 无需本地 GPU 资源
- 可扩展的批量处理能力
- 自动更新至最新模型
- 生产就绪的基础设施
- 支持异步/并发请求

## 快速入门

### 1. 获取 API 令牌

注册并获取 API 令牌：https://forge.evolutionaryscale.ai

### 2. 安装 ESM SDK

```bash
pip install esm
```

标准 ESM 软件包已包含 Forge 客户端。

### 3. 基础连接

```python
from esm.sdk.forge import ESM3ForgeInferenceClient
from esm.sdk.api import ESMProtein, GenerationConfig

# 初始化客户端
client = ESM3ForgeInferenceClient(
    model="esm3-medium-2024-08",
    url="https://forge.evolutionaryscale.ai",
    token="<your-token-here>"
)

# 测试连接
protein = ESMProtein(sequence="MPRT___KEND")
result = client.generate(protein, GenerationConfig(track="sequence", num_steps=8))
print(result.sequence)
```

## 可用模型

| 模型 ID | 参数量 | 速度 | 质量 | 适用场景 |
|----------|-----------|-------|---------|----------|
| `esm3-small-2024-08` | 14亿 | 最快 | 良好 | 快速原型设计、测试 |
| `esm3-medium-2024-08` | 70亿 | 快速 | 优秀 | 生产环境、多数应用 |
| `esm3-large-2024-03` | 980亿 | 较慢 | 最佳 | 研究、关键设计 |
| `esm3-medium-multimer-2024-09` | 70亿 | 快速 | 实验性 | 蛋白质复合体 |

**模型选择指南：**

- **开发/测试**：使用 `esm3-small-2024-08` 快速迭代
- **生产环境**：使用 `esm3-medium-2024-08` 获得最佳平衡
- **研究/关键任务**：使用 `esm3-large-2024-03` 获取最高质量
- **复合体处理**：使用 `esm3-medium-multimer-2024-09`（实验性）

## ESM3ForgeInferenceClient API

### 初始化

```python
from esm.sdk.forge import ESM3ForgeInferenceClient

# 基础初始化
client = ESM3ForgeInferenceClient(
    model="esm3-medium-2024-08",
    token="<your-token>"
)

# 自定义 URL（企业部署场景）
client = ESM3ForgeInferenceClient(
    model="esm3-medium-2024-08",
    url="https://custom.forge.instance.com",
    token="<your-token>"
)

# 配置超时时间
client = ESM3ForgeInferenceClient(
    model="esm3-medium-2024-08",
    token="<your-token>",
    timeout=300  # 5分钟
)
```

### 同步生成

标准阻塞式生成调用：

```python
from esm.sdk.api import ESMProtein, GenerationConfig

# 基础生成
protein = ESMProtein(sequence="MPRT___KEND")
config = GenerationConfig(track="sequence", num_steps=8)

result = client.generate(protein, config)
print(f"生成结果: {result.sequence}")
```

### 异步生成

支持多蛋白质并发处理：

```python
import asyncio
from esm.sdk.api import ESMProtein, GenerationConfig

async def generate_many(client, proteins):
    """并发生成多个蛋白质"""
    tasks = []

    for protein in proteins:
        task = client.async_generate(
            protein,
            GenerationConfig(track="sequence", num_steps=8)
        )
        tasks.append(task)

    results = await asyncio.gather(*tasks)
    return results

# 使用示例
proteins = [
    ESMProtein(sequence=f"MPRT{'_' * 10}KEND"),
    ESMProtein(sequence=f"AGLV{'_' * 10}HSPQ"),
    ESMProtein(sequence=f"KEIT{'_' * 10}NDFL")
]

results = asyncio.run(generate_many(client, proteins))
print(f"已生成 {len(results)} 个蛋白质")
```

### 使用 BatchExecutor 批量处理

支持自动并发管理的大规模处理：

```python
from esm.sdk.forge import BatchExecutor
from esm.sdk.api import ESMProtein, GenerationConfig

# 创建批处理执行器
executor = BatchExecutor(
    client=client,
    max_concurrent=10  # 并发处理10个请求
)

# 准备蛋白质批次
proteins = [ESMProtein(sequence=f"MPRT{'_' * 50}KEND") for _ in range(100)]
config = GenerationConfig(track="sequence", num_steps=25)

# 提交批次
batch_results = executor.submit_batch(
    proteins=proteins,
    config=config,
    progress_callback=lambda i, total: print(f"已处理 {i}/{total}")
)

print(f"完成 {len(batch_results)} 次生成")
```

## 速率限制与配额

### 理解限制规则

Forge 实施基于以下维度的速率限制：
- 每分钟请求数 (RPM)
- 每分钟令牌数 (TPM)
- 并发请求数

**典型限制（可能调整）：**
- 免费版：60 RPM，5 并发
- 专业版：300 RPM，20 并发
- 企业版：自定义限制

### 处理速率限制

```python
import time
from requests.exceptions import HTTPError

def generate_with_retry(client, protein, config, max_retries=3):
    """速率限制时自动重试"""
    for attempt in range(max_retries):
        try:
            return client.generate(protein, config)
        except HTTPError as e:
            if e.response.status_code == 429:  # 速率限制
                wait_time = 2 ** attempt  # 指数退避
                print(f"触发速率限制，等待 {wait_time}秒...")
                time.sleep(wait_time)
            else:
                raise
    raise Exception("超出最大重试次数")

# 使用示例
result = generate_with_retry(client, protein, config)
```

### 实现自定义限速器

```python
import time
from collections import deque

class RateLimiter:
    """API调用简易限速器"""

    def __init__(self, max_per_minute=60):
        self.max_per_minute = max_per_minute
        self.calls = deque()

    def wait_if_needed(self):
        """在超出速率限制时等待"""
        now = time.time()

        # 清除过期调用记录
        while self.calls and self.calls[0] < now - 60:
            self.calls.popleft()

        # 达到限制时等待
        if len(self.calls) >= self.max_per_minute:
            sleep_time = 60 - (now - self.calls[0])
            if sleep_time > 0:
                time.sleep(sleep_time)
            self.calls.popleft()

        self.calls.append(now)

# 使用示例
limiter = RateLimiter(max_per_minute=60)

for protein in proteins:
    limiter.wait_if_needed()
    result = client.generate(protein, config)
```

## 高级模式

### 流式结果处理

实时处理完成结果：

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

async def stream_generate(client, proteins, config):
    """实时返回完成结果"""
    pending = {
        asyncio.create_task(client.async_generate(p, config)): i
        for i, p in enumerate(proteins)
    }

    results = [None] * len(proteins)

    while pending:
        done, pending = await asyncio.wait(
            pending.keys(),
            return_when=asyncio.FIRST_COMPLETED
        )

        for task in done:
            idx = pending.pop(task)
            result = await task
            results[idx] = result
            yield idx, result

# 使用示例
async def process_stream():
    async for idx, result in stream_generate(client, proteins, config):
        print(f"完成蛋白质 {idx}: {result.sequence[:20]}...")

asyncio.run(process_stream())
```

### 带进度跟踪的批处理

```python
from tqdm import tqdm
import asyncio

async def batch_with_progress(client, proteins, config):
    """带进度条的批处理"""
    results = []

    with tqdm(total=len(proteins)) as pbar:
        for protein in proteins:
            result = await client.async_generate(protein, config)
            results.append(result)
            pbar.update(1)

    return results

# 使用示例
results = asyncio.run(batch_with_progress(client, proteins, config))
```

### 断点续传

适用于长时间批处理任务：

```python
import pickle
import os

class CheckpointedBatchProcessor:
    """支持断点续传的批处理器"""

    def __init__(self, client, checkpoint_file="checkpoint.pkl"):
        self.client = client
        self.checkpoint_file = checkpoint_file
        self.completed = self.load_checkpoint()

    def load_checkpoint(self):
        if os.path.exists(self.checkpoint_file):
            with open(self.checkpoint_file, 'rb') as f:
                return pickle.load(f)
        return {}

    def save_checkpoint(self):
        with open(self.checkpoint_file, 'wb') as f:
            pickle.dump(self.completed, f)

    def process_batch(self, proteins, config):
        """带断点保存的批处理"""
        results = {}

        for i, protein in enumerate(proteins):
            # 跳过已完成项
            if i in self.completed:
                results[i] = self.completed[i]
                continue

            try:
                result = self.client.generate(protein, config)
                results[i] = result
                self.completed[i] = result

                # 每处理10项保存断点
                if i % 10 == 0:
                    self.save_checkpoint()

            except Exception as e:
                print(f"处理 {i} 时出错: {e}")
                self.save_checkpoint()
                raise

        self.save_checkpoint()
        return results

# 使用示例
processor = CheckpointedBatchProcessor(client)
results = processor.process_batch(proteins, config)
```

## 错误处理

### 常见错误及解决方案

```python
from requests.exceptions import HTTPError, ConnectionError, Timeout

def robust_generate(client, protein, config):
    """带全面错误处理的生成函数"""
    try:
        return client.generate(protein, config)

    except HTTPError as e:
        if e.response.status_code == 401:
            raise ValueError("API令牌无效")
        elif e.response.status_code == 429:
            raise ValueError("超出速率限制 - 请降低请求频率")
        elif e.response.status_code == 500:
            raise ValueError("服务器错误 - 请稍后重试")
        else:
            raise

    except ConnectionError:
        raise ValueError("网络错误 - 请检查网络连接")

    except Timeout:
        raise ValueError("请求超时 - 请减小蛋白质尺寸或增加超时时间")

    except Exception as e:
        raise ValueError(f"意外错误: {str(e)}")

# 结合重试逻辑使用
def generate_with_full_retry(client, protein, config, max_retries=3):
    """错误处理与重试逻辑结合"""
    for attempt in range(max_retries):
        try:
            return robust_generate(client, protein, config)
        except ValueError as e:
            if "rate limit" in str(e).lower() and attempt < max_retries - 1:
                time.sleep(2 ** attempt)
                continue
            raise
```

## 成本优化

### 降低成本的策略

**1. 选用合适模型尺寸：**

```python
# 测试使用小型模型
dev_client = ESM3ForgeInferenceClient(
    model="esm3-small-2024-08",
    token=token
)

# 最终生成使用大型模型
prod_client = ESM3ForgeInferenceClient(
    model="esm3-large-2024-03",
    token=token
)
```

**2. 结果缓存：**

```python
import hashlib
import json

class ForgeCache:
    """本地缓存Forge API结果"""

    def __init__(self, cache_dir="forge_cache"):
        self.cache_dir = cache_dir
        os.makedirs(cache_dir, exist_ok=True)

    def get_cache_key(self, protein, config):
        """根据输入生成缓存键"""
        data = {
            'sequence': protein.sequence,
            'config': str(config)
        }
        return hashlib.md5(json.dumps(data, sort_keys=True).encode()).hexdigest()

    def get(self, protein, config):
        """获取缓存结果"""
        key = self.get_cache_key(protein, config)
        path = os.path.join(self.cache_dir, f"{key}.pkl")

        if os.path.exists(path):
            with open(path, 'rb') as f:
                return pickle.load(f)
        return None

    def set(self, protein, config, result):
        """缓存结果"""
        key = self.get_cache_key(protein, config)
        path = os.path.join(self.cache_dir, f"{key}.pkl")

        with open(path, 'wb') as f:
            pickle.dump(result, f)

# 使用示例
cache = ForgeCache()

def cached_generate(client, protein, config):
    """带缓存的生成函数"""
    cached = cache.get(protein, config)
    if cached:
        return cached

    result = client.generate(protein, config)
    cache.set(protein, config, result)
    return result
```

**3. 批量处理相似请求：**

分组处理相似任务以减少开销：

```python
def batch_similar_tasks(proteins, max_batch_size=50):
    """按相似特征分组蛋白质"""
    # 按长度排序提高处理效率
    sorted_proteins = sorted(proteins, key=lambda p: len(p.sequence))

    batches = []
    current_batch = []

    for protein in sorted_proteins:
        current_batch.append(protein)

        if len(current_batch) >= max_batch_size:
            batches.append(current_batch)
            current_batch = []

    if current_batch:
        batches.append(current_batch)

    return batches
```

## 监控与日志

### 跟踪 API 使用情况

```python
import logging
from datetime import datetime

class ForgeMonitor:
    """监控Forge API使用情况"""

    def __init__(self):
        self.calls = []
        self.errors = []

    def log_call(self, model, protein_length, duration, success=True, error=None):
        """记录API调用"""
        entry = {
            'timestamp': datetime.now(),
            'model': model,
            'protein_length': protein_length,
            'duration': duration,
            'success': success,
            'error': str(error) if error else None
        }

        if success:
            self.calls.append(entry)
        else:
            self.errors.append(entry)

    def get_stats(self):
        """获取使用统计"""
        total_calls = len(self.calls) + len(self.errors)
        success_rate = len(self.calls) / total_calls if total_calls > 0 else 0
        avg_duration = sum(c['duration'] for c in self.calls) / len(self.calls) if self.calls else 0

        return {
            'total_calls': total_calls,
            'successful': len(self.calls),
            'failed': len(self.errors),
            'success_rate': success_rate,
            'avg_duration': avg_duration
        }

# 使用示例
monitor = ForgeMonitor()

def monitored_generate(client, protein, config):
    """带监控的生成函数"""
    start = time.time()

    try:
        result = client.generate(protein, config)
        duration = time.time() - start
        monitor.log_call(
            model=client.model,
            protein_length=len(protein.sequence),
            duration=duration,
            success=True
        )
        return result

    except Exception as e:
        duration = time.time() - start
        monitor.log_call(
            model=client.model,
            protein_length=len(protein.sequence),
            duration=duration,
            success=False,
            error=e
        )
        raise

# 查看统计
print(monitor.get_stats())
```

## AWS SageMaker 部署

适用于专用基础设施和企业场景：

### 部署选项

1. **AWS Marketplace 部署**：通过 AWS SageMaker Marketplace 部署 ESM3
2. **自定义端点**：配置专用推理端点
3. **批量转换**：使用 SageMaker Batch Transform 进行大规模处理

### 优势

- 专属计算资源
- 无基础设施外速率限制
- 数据保留在 AWS 环境内
- 与 AWS 服务集成
- 自定义实例类型与扩展能力

**更多信息：**
- AWS Marketplace：https://aws.amazon.com/marketplace/seller-profile?id=seller-iw2nbs

2. **速率限制**：实现指数退避并遵守限制  
3. **错误处理**：始终处理网络错误和重试机制  
4. **缓存**：对重复查询结果进行缓存  
5. **模型选择**：根据任务选用合适规模的模型  
6. **批处理**：对多蛋白质任务使用异步/批处理  
7. **监控**：跟踪使用情况和成本  
8. **检查点**：为长时间任务保存进度  

## 故障排除  

### 连接问题  

```python  
# 测试连接  
try:  
    client = ESM3ForgeInferenceClient(model="esm3-medium-2024-08", token=token)  
    test_protein = ESMProtein(sequence="MPRTK")  
    result = client.generate(test_protein, GenerationConfig(track="sequence", num_steps=1))  
    print("连接成功！")  
except Exception as e:  
    print(f"连接失败: {e}")  
```  

### 令牌验证  

```python  
def validate_token(token):  
    """验证API令牌"""  
    try:  
        client = ESM3ForgeInferenceClient(  
            model="esm3-small-2024-08",  
            token=token  
        )  
        # 执行最小化测试调用  
        test = ESMProtein(sequence="MPR")  
        client.generate(test, GenerationConfig(track="sequence", num_steps=1))  
        return True  
    except HTTPError as e:  
        if e.response.status_code == 401:  
            return False  
        raise  
```  

## 附加资源  

- **Forge平台**：https://forge.evolutionaryscale.ai  
- **API文档**：查看Forge仪表板获取最新API规范  
- **社区支持**：Slack社区 https://bit.ly/3FKwcWd  
- **企业联系**：联系EvolutionaryScale获取定制化部署
