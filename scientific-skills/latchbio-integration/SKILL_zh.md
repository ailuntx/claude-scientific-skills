---
name: latchbio-integration
description: 面向生物信息学工作流的Latch平台。使用Latch SDK构建流程，通过@workflow/@task装饰器部署无服务器工作流，支持LatchFile/LatchDir数据管理及Nextflow/Snakemake集成。
license: Unknown
metadata:
    skill-author: K-Dense Inc.
---

# LatchBio 集成方案

## 概述

Latch 是一个用于构建和部署生物信息学工作流的Python框架，可将其转化为无服务器流程。基于Flyte构建，支持通过@workflow/@task装饰器创建工作流，使用LatchFile/LatchDir管理云数据，配置计算资源，并集成Nextflow/Snakemake流程。

## 核心能力

Latch平台提供四大核心功能：

### 1. 工作流创建与部署
- 使用Python装饰器定义无服务器工作流
- 支持原生Python、Nextflow和Snakemake流程
- 通过Docker自动容器化
- 自动生成无代码用户界面
- 版本控制与可复现性

### 2. 数据管理
- 云存储抽象层（LatchFile, LatchDir）
- 通过注册表实现结构化数据管理（项目→表格→记录）
- 类型安全的数据操作（链接与枚举）
- 本地与云端自动文件传输
- 通配符模式匹配文件选择

### 3. 资源配置
- 预配置任务装饰器（@small_task, @large_task, @small_gpu_task, @large_gpu_task）
- 自定义资源规格（CPU/内存/GPU/存储）
- GPU支持（K80/V100/A100）
- 超时与存储配置
- 成本优化策略

### 4. 已验证工作流
- 生产级预构建流程
- 批量RNA-seq、DESeq2、通路分析
- 蛋白质结构预测（AlphaFold/ColabFold）
- 单细胞工具（ArchR/scVelo/emptyDropsR）
- CRISPR分析、系统发育学等

## 快速入门

### 安装与设置

```bash
# 安装Latch SDK
python3 -m uv pip install latch

# 登录Latch平台
latch login

# 初始化新工作流
latch init my-workflow

# 注册工作流至平台
latch register my-workflow
```

**前提条件：**
- 已安装并运行Docker
- 拥有Latch账户凭证
- Python 3.8+

### 基础工作流示例

```python
from latch import workflow, small_task
from latch.types import LatchFile

@small_task
def process_file(input_file: LatchFile) -> LatchFile:
    """处理单个文件"""
    # 处理逻辑
    return output_file

@workflow
def my_workflow(input_file: LatchFile) -> LatchFile:
    """
    生物信息学工作流
    
    参数:
        input_file: 输入数据文件
    """
    return process_file(input_file=input_file)
```

## 适用场景

当遇到以下情况时使用本技能：

**工作流开发：**
- "创建RNA-seq分析的Latch工作流"
- "将我的流程部署到Latch平台"
- "将Nextflow流程转换为Latch格式"
- "为工作流添加GPU支持"
- 使用`@workflow`、`@task`装饰器

**数据管理：**
- "在Latch注册表中组织测序数据"
- "如何使用LatchFile和LatchDir？"
- "在Latch中设置样本追踪"
- 使用`latch:///`路径

**资源配置：**
- "在Latch上为AlphaFold配置GPU"
- "任务内存不足的解决方案"
- "如何优化工作流成本？"
- 使用任务装饰器

**已验证工作流：**
- "在Latch上运行AlphaFold"
- "使用DESeq2进行差异表达分析"
- "可用的预构建工作流"
- 使用`latch.verified`模块

## 详细文档

本技能包含按功能分类的完整参考文档：

### references/workflow-creation.md
**阅读场景：**
- 创建与注册工作流
- 任务定义与装饰器
- 支持Python/Nextflow/Snakemake
- 启动计划与条件分支
- 工作流执行（CLI与编程接口）
- 多步骤与并行流程
- 注册问题排查

**核心主题：**
- `latch init`和`latch register`命令
- `@workflow`和`@task`装饰器
- LatchFile/LatchDir基础
- 类型注解与文档字符串
- 预设参数启动计划
- 条件化UI区块

### references/data-management.md
**阅读场景：**
- LatchFile/LatchDir云存储
- 注册表系统（项目/表格/记录）
- 链接记录与关系管理
- 枚举与类型化列
- 批量操作与事务处理
- 与工作流集成
- 账户与工作区管理

**核心主题：**
- `latch:///`路径格式
- 文件传输与通配符匹配
- 创建/查询注册表
- 列类型（字符串/数字/文件/链接/枚举）
- 记录增删改查操作
- 工作流-注册表集成

### references/resource-configuration.md
**阅读场景：**
- 任务资源装饰器
- 自定义CPU/内存/GPU配置
- GPU类型（K80/V100/A100）
- 超时与存储设置
- 资源优化策略
- 成本效益设计
- 监控与调试

**核心主题：**
- `@small_task`、`@large_task`、`@small_gpu_task`、`@large_gpu_task`
- 精确规格的`@custom_task`
- 多GPU配置
- 按负载类型选择资源
- 平台限制与配额

### references/verified-workflows.md
**阅读场景：**
- 预构建生产工作流
- 批量RNA-seq与DESeq2
- AlphaFold与ColabFold
- 单细胞分析（ArchR/scVelo）
- CRISPR编辑分析
- 通路富集分析
- 与自定义工作流集成

**核心主题：**
- `latch.verified`模块导入
- 可用已验证工作流
- 工作流参数与选项
- 组合验证与自定义步骤
- 版本管理

## 典型工作流模式

### 完整RNA-seq流程

```python
from latch import workflow, small_task, large_task
from latch.types import LatchFile, LatchDir

@small_task
def quality_control(fastq: LatchFile) -> LatchFile:
    """运行FastQC"""
    return qc_output

@large_task
def alignment(fastq: LatchFile, genome: str) -> LatchFile:
    """STAR比对"""
    return bam_output

@small_task
def quantification(bam: LatchFile) -> LatchFile:
    """featureCounts定量"""
    return counts

@workflow
def rnaseq_pipeline(
    input_fastq: LatchFile,
    genome: str,
    output_dir: LatchDir
) -> LatchFile:
    """RNA-seq分析流程"""
    qc = quality_control(fastq=input_fastq)
    aligned = alignment(fastq=qc, genome=genome)
    return quantification(bam=aligned)
```

### GPU加速工作流

```python
from latch import workflow, small_task, large_gpu_task
from latch.types import LatchFile

@small_task
def preprocess(input_file: LatchFile) -> LatchFile:
    """数据预处理"""
    return processed

@large_gpu_task
def gpu_computation(data: LatchFile) -> LatchFile:
    """GPU加速分析"""
    return results

@workflow
def gpu_pipeline(input_file: LatchFile) -> LatchFile:
    """含GPU任务的工作流"""
    preprocessed = preprocess(input_file=input_file)
    return gpu_computation(data=preprocessed)
```

### 注册表集成工作流

```python
from latch import workflow, small_task
from latch.registry.table import Table
from latch.registry.record import Record
from latch.types import LatchFile

@small_task
def process_and_track(sample_id: str, table_id: str) -> str:
    """处理样本并更新注册表"""
    # 从注册表获取样本
    table = Table.get(table_id=table_id)
    records = Record.list(table_id=table_id, filter={"sample_id": sample_id})
    sample = records[0]

    # 处理流程
    input_file = sample.values["fastq_file"]
    output = process(input_file)

    # 更新注册表
    sample.update(values={"status": "completed", "result": output})
    return "Success"

@workflow
def registry_workflow(sample_id: str, table_id: str):
    """与注册表集成的工作流"""
    return process_and_track(sample_id=sample_id, table_id=table_id)
```

## 最佳实践

### 工作流设计
1. 为所有参数添加类型注解
2. 编写清晰文档字符串（将显示在UI中）
3. 从标准任务装饰器起步，按需扩展
4. 将复杂工作流拆分为模块化任务
5. 实现完善的错误处理

### 数据管理
6. 使用一致的文件夹结构
7. 批量录入前定义注册表模式
8. 使用链接记录管理关联关系
9. 在注册表中存储元数据确保可追溯性

### 资源配置
10. 合理分配资源（避免过度配置）
11. 仅在算法支持时启用GPU
12. 监控执行指标并持续优化
13. 尽可能设计并行执行

### 开发流程
14. 注册前使用Docker本地测试
15. 对工作流代码进行版本控制
16. 文档化资源需求
17. 性能剖析确定实际需求

## 故障排查

### 常见问题

**注册失败：**
- 确保Docker正在运行
- 通过`latch login`检查认证状态
- 验证Dockerfile中所有依赖项
- 使用`--verbose`标志获取详细日志

**资源问题：**
- 内存不足：增加任务装饰器的内存配置
- 超时：延长超时参数
- 存储问题：增加临时存储空间（storage_gib）

**数据访问：**
- 使用正确的`latch:///`路径格式
- 确认文件存在于工作区
- 检查共享工作区权限

**类型错误：**
- 为所有参数添加类型注解
- 文件/目录参数使用LatchFile/LatchDir类型
- 确保工作流返回类型与实际匹配

## 扩展资源

- **官方文档**: https://docs.latch.bio
- **GitHub仓库**: https://github.com/latchbio/latch
- **Slack社区**: 加入Latch SDK工作区
- **API参考**: https://docs.latch.bio/api/latch.html
- **技术博客**: https://blog.latch.bio

## 技术支持

遇到问题时可参考：
1. 查阅上述文档链接
2. 搜索GitHub issues
3. 在Slack社区提问
4. 联系 support@latch.bio
