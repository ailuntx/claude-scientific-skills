# 工作流创建与注册

## 概述
Latch SDK 支持使用 Python 装饰器定义无服务器生物信息学工作流，并通过自动容器化和 UI 生成进行部署。

## 安装

安装 Latch SDK：
```bash
python3 -m pip install latch
```

**前提条件：**
- 必须本地安装并运行 Docker
- 需拥有 Latch 账户凭证

## 初始化新工作流

创建工作流模板：
```bash
latch init <工作流名称>
```

该命令将生成包含以下内容的工作流目录：
- `wf/__init__.py` - 主工作流定义文件
- `Dockerfile` - 容器配置文件
- `version` - 版本追踪文件

## 工作流定义结构

### 基础工作流示例

```python
from latch import workflow
from latch.types import LatchFile, LatchDir

@workflow
def my_workflow(input_file: LatchFile, output_dir: LatchDir) -> LatchFile:
    """
    显示在UI中的工作流描述

    Args:
        input_file: 输入文件描述
        output_dir: 输出目录描述
    """
    return process_task(input_file, output_dir)
```

### 任务定义

任务是工作流中的独立计算单元：

```python
from latch import small_task, large_task

@small_task
def process_task(input_file: LatchFile, output_dir: LatchDir) -> LatchFile:
    """任务级计算"""
    # 此处为处理逻辑
    return output_file
```

### 任务资源装饰器

SDK 提供多种资源需求的任务装饰器：

- `@small_task` - 轻量级任务默认资源
- `@large_task` - 增强内存与CPU
- `@small_gpu_task` - 最小资源的GPU任务
- `@large_gpu_task` - 最大资源的GPU任务
- `@custom_task` - 自定义资源配置

## 注册工作流

向 Latch 平台注册工作流：
```bash
latch register <工作流目录>
```

注册流程：
1. 构建包含所有依赖的 Docker 容器
2. 序列化工作流代码
3. 上传容器至注册中心
4. 自动生成无代码 UI
5. 使工作流在平台可用

### 注册输出

成功注册后：
- 工作流出现在 Latch 工作空间
- 自动生成带参数表单的 UI
- 版本被追踪并容器化
- 可立即执行工作流

## 支持多语言流程

Latch 支持上传以下语言的现有流程：
- **Python** - 原生 Latch SDK 工作流
- **Nextflow** - 导入现有 Nextflow 流程
- **Snakemake** - 导入现有 Snakemake 流程

### Nextflow 集成

导入 Nextflow 流程：
```bash
latch register --nextflow <nextflow目录>
```

### Snakemake 集成

导入 Snakemake 流程：
```bash
latch register --snakemake <snakemake目录>
```

## 工作流执行

### 通过命令行

执行已注册工作流：
```bash
latch execute <工作流名称> --input-file <路径> --output-dir <路径>
```

### 通过 Python

编程方式执行工作流：
```python
from latch.account import Account
from latch.executions import execute_workflow

account = Account.current()
execution = execute_workflow(
    workflow_name="my_workflow",
    parameters={
        "input_file": "/path/to/file",
        "output_dir": "/path/to/output"
    }
)
```

## 启动计划

启动计划定义预设参数配置：

```python
from latch.resources.launch_plan import LaunchPlan

# 创建带预设参数的启动计划
launch_plan = LaunchPlan.create(
    workflow_name="my_workflow",
    name="default_config",
    default_inputs={
        "input_file": "/data/sample.fastq",
        "output_dir": "/results"
    }
)
```

## 条件参数区块

创建动态 UI 的条件参数区块：

```python
from latch.types import LatchParameter
from latch.resources.conditional import conditional_section

@workflow
def my_workflow(
    mode: str,
    advanced_param: str = conditional_section(
        condition=lambda inputs: inputs.mode == "advanced"
    )
):
    """带条件参数的工作流"""
    pass
```

## 最佳实践

1. **类型标注**：始终为工作流参数添加类型提示
2. **文档字符串**：提供清晰文档 - 它们将填充 UI 描述
3. **版本控制**：使用语义化版本管理工作流更新
4. **测试**：注册前在本地测试工作流
5. **资源规划**：从较小资源装饰器开始，按需扩展
6. **模块化设计**：将复杂工作流拆分为可复用任务
7. **错误处理**：在任务中实现恰当的错误处理
8. **日志记录**：使用 Python 日志进行调试和监控

## 常用模式

### 多步骤流程

```python
from latch import workflow, small_task
from latch.types import LatchFile

@small_task
def quality_control(input_file: LatchFile) -> LatchFile:
    """质控步骤"""
    return qc_output

@small_task
def alignment(qc_file: LatchFile) -> LatchFile:
    """比对步骤"""
    return aligned_output

@workflow
def rnaseq_pipeline(input_fastq: LatchFile) -> LatchFile:
    """RNA-seq分析流程"""
    qc_result = quality_control(input_file=input_fastq)
    aligned = alignment(qc_file=qc_result)
    return aligned
```

### 并行处理

```python
from typing import List
from latch import workflow, small_task, map_task
from latch.types import LatchFile

@small_task
def process_sample(sample: LatchFile) -> LatchFile:
    """处理单个样本"""
    return processed_sample

@workflow
def batch_pipeline(samples: List[LatchFile]) -> List[LatchFile]:
    """并行处理多个样本"""
    return map_task(process_sample)(sample=samples)
```

## 故障排除

### 常见问题

1. **Docker未运行**：确保 Docker 守护进程处于活动状态
2. **认证错误**：运行 `latch login` 刷新凭证
3. **构建失败**：检查 Dockerfile 是否缺少依赖
4. **类型错误**：确保所有参数都有正确的类型标注

### 调试模式

注册时启用详细日志：
```bash
latch register --verbose <工作流目录>
```

## 参考资源

- 官方文档：https://docs.latch.bio
- GitHub 仓库：https://github.com/latchbio/latch
- Slack 社区：https://join.slack.com/t/latchbiosdk
