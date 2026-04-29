# 资源配置

## 概述
Latch SDK 为工作流任务提供灵活的资源配置功能，支持在包括 CPU、GPU 和内存优化实例在内的合适计算基础设施上高效执行。

## 任务资源装饰器

### 标准装饰器

SDK 为常见资源需求提供预配置的任务装饰器：

#### @small_task
轻量级任务的默认配置：
```python
from latch import small_task

@small_task
def lightweight_processing():
    """最小资源需求"""
    pass
```

**适用场景：**
- 文件解析与操作
- 简单数据转换
- 快速质控检查
- 元数据操作

#### @large_task
增强型 CPU 和内存配置，适用于计算密集型任务：
```python
from latch import large_task

@large_task
def intensive_computation():
    """更高的 CPU 和内存分配"""
    pass
```

**适用场景：**
- 大文件处理
- 复杂统计分析
- 序列组装任务
- 多线程操作

#### @small_gpu_task
基础资源的 GPU 支持任务：
```python
from latch import small_gpu_task

@small_gpu_task
def gpu_inference():
    """基础资源的 GPU 任务"""
    pass
```

**适用场景：**
- 神经网络推理
- 小规模机器学习预测
- GPU 加速库操作

#### @large_gpu_task
最大化资源的 GPU 支持任务：
```python
from latch import large_gpu_task

@large_gpu_task
def gpu_training():
    """最大化 CPU 和内存的 GPU 任务"""
    pass
```

**适用场景：**
- 深度学习模型训练
- 蛋白质结构预测 (AlphaFold)
- 大规模 GPU 计算

### 自定义任务配置

如需精确控制，使用 `@custom_task` 装饰器：

```python
from latch import custom_task
from latch.resources.tasks import TaskResources

@custom_task(
    cpu=8,
    memory=32,  # GB
    storage_gib=100,
    timeout=3600,  # 秒
)
def custom_processing():
    """自定义资源配置的任务"""
    pass
```

#### 自定义任务参数

- **cpu**: CPU 核心数 (整数)
- **memory**: 内存大小 (GB，整数)
- **storage_gib**: 临时存储空间 (GiB，整数)
- **timeout**: 最大执行时间 (秒，整数)
- **gpu**: GPU 数量 (整数，0 表示仅 CPU)
- **gpu_type**: 指定 GPU 型号 (字符串，如 "nvidia-tesla-v100")

### 高级自定义配置

```python
from latch.resources.tasks import TaskResources

@custom_task(
    cpu=16,
    memory=64,
    storage_gib=500,
    timeout=7200,
    gpu=1,
    gpu_type="nvidia-tesla-a100"
)
def alphafold_prediction():
    """A100 GPU 和高内存的 AlphaFold 任务"""
    pass
```

## GPU 配置

### GPU 类型

可用 GPU 选项：
- **nvidia-tesla-k80**: 基础测试用 GPU
- **nvidia-tesla-v100**: 高性能训练 GPU
- **nvidia-tesla-a100**: 最新一代顶级性能 GPU

### 多 GPU 任务

```python
from latch import custom_task

@custom_task(
    cpu=32,
    memory=128,
    gpu=4,
    gpu_type="nvidia-tesla-v100"
)
def multi_gpu_training():
    """跨多 GPU 的分布式训练"""
    pass
```

## 资源选择策略

### 按计算需求

**内存密集型任务：**
```python
@custom_task(cpu=4, memory=128)  # 高内存，中等 CPU
def genome_assembly():
    pass
```

**CPU 密集型任务：**
```python
@custom_task(cpu=64, memory=32)  # 高 CPU，中等内存
def parallel_alignment():
    pass
```

**I/O 密集型任务：**
```python
@custom_task(cpu=8, memory=16, storage_gib=1000)  # 大容量临时存储
def data_preprocessing():
    pass
```

### 按工作流阶段

**快速验证：**
```python
@small_task
def validate_inputs():
    """快速输入验证"""
    pass
```

**主计算：**
```python
@large_task
def primary_analysis():
    """资源密集型分析"""
    pass
```

**结果汇总：**
```python
@small_task
def aggregate_results():
    """轻量级结果汇总"""
    pass
```

## 工作流资源规划

### 完整流程示例

```python
from latch import workflow, small_task, large_task, large_gpu_task
from latch.types import LatchFile

@small_task
def quality_control(fastq: LatchFile) -> LatchFile:
    """质控无需过多资源"""
    return qc_output

@large_task
def alignment(fastq: LatchFile) -> LatchFile:
    """序列比对受益于更多 CPU"""
    return bam_output

@large_gpu_task
def variant_calling(bam: LatchFile) -> LatchFile:
    """GPU 加速的变异检测"""
    return vcf_output

@small_task
def generate_report(vcf: LatchFile) -> LatchFile:
    """简单报告生成"""
    return report

@workflow
def genomics_pipeline(input_fastq: LatchFile) -> LatchFile:
    """资源优化的基因组学流程"""
    qc = quality_control(fastq=input_fastq)
    aligned = alignment(fastq=qc)
    variants = variant_calling(bam=aligned)
    return generate_report(vcf=variants)
```

## 超时配置

### 设置超时

```python
from latch import custom_task

@custom_task(
    cpu=8,
    memory=32,
    timeout=10800  # 3 小时（秒）
)
def long_running_analysis():
    """延长超时的分析任务"""
    pass
```

### 超时最佳实践

1. **保守预估**：在预期时长基础上增加缓冲时间
2. **监控实际运行**：根据执行数据调整
3. **默认超时**：多数任务默认 1 小时
4. **最大超时**：检查平台对长时任务的限制

## 存储配置

### 临时存储

为中间文件配置临时存储：

```python
@custom_task(
    cpu=8,
    memory=32,
    storage_gib=500  # 500 GB 临时存储
)
def process_large_dataset():
    """含大型中间文件的任务"""
    # 临时存储路径 /tmp
    temp_file = "/tmp/intermediate_data.bam"
    pass
```

### 存储指南

- 默认存储通常满足多数任务需求
- 含大型中间文件的任务需指定更大存储
- 任务完成后临时存储会被清除
- 持久存储需求请使用 LatchDir

## 成本优化

### 资源效率技巧

1. **合理分配**：避免过度配置
2. **选用合适装饰器**：从标准装饰器开始
3. **按需使用 GPU**：GPU 任务成本更高
4. **并行小任务**：优于单个大任务
5. **监控使用**：审查实际资源利用率

### 示例：成本优化设计

```python
# 低效：所有任务使用大资源
@large_task
def validate_input():  # 资源过剩
    pass

@large_task
def simple_transformation():  # 资源过剩
    pass

# 高效：合理分配资源
@small_task
def validate_input():  # 合理配置
    pass

@small_task
def simple_transformation():  # 合理配置
    pass

@large_task
def intensive_analysis():  # 合理配置
    pass
```

## 监控与调试

### 资源使用监控

工作流执行期间监控：
- CPU 利用率
- 内存使用
- GPU 利用率（如适用）
- 执行时长
- 存储消耗

### 常见资源问题

**内存不足 (OOM)：**
```python
# 解决方案：增加内存分配
@custom_task(cpu=8, memory=64)  # 从 32GB 增至 64GB
def memory_intensive_task():
    pass
```

**超时：**
```python
# 解决方案：延长超时
@custom_task(cpu=8, memory=32, timeout=14400)  # 4 小时
def long_task():
    pass
```

**存储不足：**
```python
# 解决方案：增加临时存储
@custom_task(cpu=8, memory=32, storage_gib=1000)
def large_intermediate_files():
    pass
```

## 条件资源配置

根据输入动态分配资源：

```python
from latch import workflow, custom_task
from latch.types import LatchFile

def get_resource_config(file_size_gb: float):
    """根据文件大小确定资源"""
    if file_size_gb < 10:
        return {"cpu": 4, "memory": 16}
    elif file_size_gb < 100:
        return {"cpu": 16, "memory": 64}
    else:
        return {"cpu": 32, "memory": 128}

# 注意：资源装饰器需静态定义
# 不同规模使用不同任务变体
@custom_task(cpu=4, memory=16)
def process_small(file: LatchFile) -> LatchFile:
    pass

@custom_task(cpu=16, memory=64)
def process_medium(file: LatchFile) -> LatchFile:
    pass

@custom_task(cpu=32, memory=128)
def process_large(file: LatchFile) -> LatchFile:
    pass
```

## 最佳实践总结

1. **从小开始**：先用标准装饰器，按需扩展
2. **性能分析**：通过测试运行确定实际需求
3. **谨慎使用 GPU**：仅在算法支持时启用
4. **并行设计**：尽可能拆分为小任务
5. **监控调整**：审查执行指标并优化
6. **文档说明**：注释资源需求的设定原因
7. **本地测试**：注册前使用 Docker 本地验证
8. **成本考量**：平衡性能与成本效益

## 平台特定考量

### 可用资源

Latch 平台提供：
- CPU 实例：最高 96 核
- 内存：最高 768 GB
- GPU：K80/V100/A100 等型号
- 存储：可配置临时存储

### 资源限制

查看当前平台限制：
- 单任务最大 CPU
- 单任务最大内存
- 最大 GPU 分配量
- 最大并发任务数

### 配额与限制

注意工作区配额：
- 总并发执行数
- 总资源分配量
- 存储限制
- 执行时间限制

如需提升配额请联系 Latch 技术支持。
