---
name: dnanexus-integration
description: DNAnexus 云基因组学平台。构建应用/小程序，管理数据（上传/下载），dxpy Python SDK，运行工作流，处理 FASTQ/BAM/VCF 文件，用于基因组学流程开发与执行。
license: 未知
compatibility: 需要 DNAnexus 账户
metadata:
    skill-author: K-Dense 公司
---

# DNAnexus 集成

## 概述

DNAnexus 是面向生物医学数据分析和基因组学的云平台。可构建部署应用/小程序，管理数据对象，运行工作流，并使用 dxpy Python SDK 进行基因组学流程开发与执行。

## 使用场景

本技能适用于以下场景：
- 创建、构建或修改 DNAnexus 应用/小程序
- 上传、下载、搜索或整理文件与记录
- 运行分析、监控任务、创建工作流
- 使用 dxpy 编写脚本与平台交互
- 配置 dxapp.json、管理依赖项、使用 Docker
- 处理 FASTQ、BAM、VCF 等生物信息学文件
- 管理项目、权限或平台资源

## 核心能力

本技能分为五大功能模块，均提供详细参考文档：

### 1. 应用开发

**目的**：创建可在 DNAnexus 平台运行的可执行程序（应用/小程序）。

**关键操作**：
- 使用 `dx-app-wizard` 生成应用骨架
- 编写带正确入口点的 Python/Bash 应用
- 处理输入/输出数据对象
- 通过 `dx build` 或 `dx build --app` 部署
- 在平台上测试应用

**典型用例**：
- 生物信息学流程（比对、变异检测）
- 数据处理工作流
- 质量控制与过滤
- 格式转换工具

**参考**：查阅 `references/app-development.md` 获取：
- 完整应用结构与模式
- Python 入口点装饰器
- dxpy 输入输出处理
- 开发最佳实践
- 常见问题解决方案

### 2. 数据操作

**目的**：管理平台上的文件、记录等数据对象。

**关键操作**：
- 通过 `dxpy.upload_local_file()` 和 `dxpy.download_dxfile()` 上传/下载文件
- 创建带元数据的记录
- 按名称/属性/类型搜索数据对象
- 跨项目克隆数据
- 管理项目文件夹与权限

**典型用例**：
- 上传测序数据（FASTQ 文件）
- 整理分析结果
- 搜索特定样本或实验
- 跨项目数据备份
- 管理参考基因组与注释

**参考**：查阅 `references/data-operations.md` 获取：
- 完整文件与记录操作
- 数据对象生命周期（开启/关闭状态）
- 搜索与发现模式
- 项目管理
- 批量操作

### 3. 任务执行

**目的**：运行分析、监控执行并编排工作流。

**关键操作**：
- 通过 `applet.run()` 或 `app.run()` 启动任务
- 监控任务状态与日志
- 创建并行处理的子任务
- 构建运行多步骤工作流
- 通过输出引用串联任务

**典型用例**：
- 对测序数据运行基因组学分析
- 多样本并行处理
- 多步骤分析流程
- 监控长时计算任务
- 调试失败任务

**参考**：查阅 `references/job-execution.md` 获取：
- 完整任务生命周期与状态
- 工作流创建与编排
- 并行执行模式
- 任务监控与调试
- 资源管理

### 4. Python SDK (dxpy)

**目的**：通过 Python 编程访问 DNAnexus 平台。

**关键操作**：
- 使用数据对象句柄（DXFile、DXRecord、DXApplet 等）
- 调用高级函数处理常规任务
- 直接发起 API 调用执行高级操作
- 创建对象间链接与引用
- 搜索发现平台资源

**典型用例**：
- 数据管理自动化脚本
- 定制分析流程
- 批量处理工作流
- 外部工具集成
- 数据迁移与整理

**参考**：查阅 `references/python-sdk.md` 获取：
- 完整 dxpy 类参考
- 高级工具函数
- API 方法文档
- 错误处理模式
- 常用代码模式

### 5. 配置与依赖

**目的**：配置应用元数据并管理依赖项。

**关键操作**：
- 编写含输入/输出和运行规范的 dxapp.json
- 安装系统包（execDepends）
- 打包定制工具与资源
- 使用共享依赖资源（assets）
- 集成 Docker 容器
- 配置实例类型与超时设置

**典型用例**：
- 定义应用输入/输出规范
- 安装生物信息学工具（samtools、bwa 等）
- 管理 Python 包依赖
- 使用 Docker 镜像构建复杂环境
- 选择计算资源

**参考**：查阅 `references/configuration.md` 获取：
- 完整 dxapp.json 规范
- 依赖管理策略
- Docker 集成模式
- 区域与资源配置
- 配置示例

## 快速入门示例

### 上传并分析数据

```python
import dxpy

# 上传输入文件
input_file = dxpy.upload_local_file("sample.fastq", project="project-xxxx")

# 运行分析
job = dxpy.DXApplet("applet-xxxx").run({
    "reads": dxpy.dxlink(input_file.get_id())
})

# 等待完成
job.wait_on_done()

# 下载结果
output_id = job.describe()["output"]["aligned_reads"]["$dnanexus_link"]
dxpy.download_dxfile(output_id, "aligned.bam")
```

### 搜索并下载文件

```python
import dxpy

# 查找特定实验的BAM文件
files = dxpy.find_data_objects(
    classname="file",
    name="*.bam",
    properties={"experiment": "exp001"},
    project="project-xxxx"
)

# 下载每个文件
for file_result in files:
    file_obj = dxpy.DXFile(file_result["id"])
    filename = file_obj.describe()["name"]
    dxpy.download_dxfile(file_result["id"], filename)
```

### 创建简单应用

```python
# src/my-app.py
import dxpy
import subprocess

@dxpy.entry_point('main')
def main(input_file, quality_threshold=30):
    # 下载输入
    dxpy.download_dxfile(input_file["$dnanexus_link"], "input.fastq")

    # 处理
    subprocess.check_call([
        "quality_filter",
        "--input", "input.fastq",
        "--output", "filtered.fastq",
        "--threshold", str(quality_threshold)
    ])

    # 上传输出
    output_file = dxpy.upload_local_file("filtered.fastq")

    return {
        "filtered_reads": dxpy.dxlink(output_file)
    }

dxpy.run()
```

## 工作流决策树

使用 DNAnexus 时遵循以下决策流程：

1. **需创建新可执行程序？**
   - 是 → 使用**应用开发** (references/app-development.md)
   - 否 → 进入第2步

2. **需管理文件或数据？**
   - 是 → 使用**数据操作** (references/data-operations.md)
   - 否 → 进入第3步

3. **需运行分析或工作流？**
   - 是 → 使用**任务执行** (references/job-execution.md)
   - 否 → 进入第4步

4. **需编写自动化Python脚本？**
   - 是 → 使用**Python SDK** (references/python-sdk.md)
   - 否 → 进入第5步

5. **需配置应用设置或依赖？**
   - 是 → 使用**配置管理** (references/configuration.md)

通常需组合多项能力（如应用开发+配置管理，或数据操作+任务执行）。

## 安装与认证

### 安装 dxpy

```bash
uv pip install dxpy
```

### 登录 DNAnexus

```bash
dx login
```

此操作将认证会话并设置项目与数据访问权限。

### 验证安装

```bash
dx --version
dx whoami
```

## 常用模式

### 模式1：批量处理

使用相同分析处理多个文件：

```python
# 查找所有FASTQ文件
files = dxpy.find_data_objects(
    classname="file",
    name="*.fastq",
    project="project-xxxx"
)

# 启动并行任务
jobs = []
for file_result in files:
    job = dxpy.DXApplet("applet-xxxx").run({
        "input": dxpy.dxlink(file_result["id"])
    })
    jobs.append(job)

# 等待所有任务完成
for job in jobs:
    job.wait_on_done()
```

### 模式2：多步骤流程

串联多个分析任务：

```python
# 步骤1：质量控制
qc_job = qc_applet.run({"reads": input_file})

# 步骤2：序列比对（使用QC输出）
align_job = align_applet.run({
    "reads": qc_job.get_output_ref("filtered_reads")
})

# 步骤3：变异检测（使用比对输出）
variant_job = variant_applet.run({
    "bam": align_job.get_output_ref("aligned_bam")
})
```

### 模式3：数据组织

系统化整理分析结果：

```python
# 创建结构化文件夹
dxpy.api.project_new_folder(
    "project-xxxx",
    {"folder": "/experiments/exp001/results", "parents": True}
)

# 带元数据上传
result_file = dxpy.upload_local_file(
    "results.txt",
    project="project-xxxx",
    folder="/experiments/exp001/results",
    properties={
        "experiment": "exp001",
        "sample": "sample1",
        "analysis_date": "2025-10-20"
    },
    tags=["validated", "published"]
)
```

## 最佳实践

1. **错误处理**：始终用 try-except 包裹 API 调用
2. **资源管理**：为工作负载选择合适的实例类型
3. **数据组织**：采用一致的文件夹结构与元数据
4. **成本优化**：归档旧数据，选用适当存储类型
5. **文档规范**：在 dxapp.json 中包含清晰描述
6. **测试验证**：生产前使用多类型输入测试应用
7. **版本控制**：对应用采用语义化版本管理
8. **安全准则**：避免在源码硬编码凭证
9. **日志记录**：包含可调试的日志信息
10. **清理机制**：移除临时文件与失败任务

## 资源

本技能包含详细参考文档：

### references/

- **app-development.md** - 应用/小程序构建部署完整指南
- **data-operations.md** - 文件管理、记录、搜索与项目操作
- **job-execution.md** - 任务运行、工作流、监控与并行处理
- **python-sdk.md** - 含所有类与函数的 dxpy 库完整参考
- **configuration.md** - dxapp.json 规范与依赖管理

处理复杂任务或需要特定操作细节时，请加载对应参考文档。

## 获取帮助

- 官方文档：https://documentation.dnanexus.com/
- API 参考：http://autodoc.dnanexus.com/
- GitHub 仓库：https://github.com/dnanexus/dx-toolkit
- 技术支持：support@dnanexus.com
