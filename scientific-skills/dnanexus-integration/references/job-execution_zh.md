# DNAnexus 作业执行与工作流

## 概述

作业是 DNAnexus 平台上的基本执行单元。当应用或小程序运行时，系统会在隔离的 Linux 环境中创建作业并在工作节点上执行，该环境提供持续的 API 访问能力。

## 作业类型

### 初始作业 (Origin Jobs)
由用户或自动化系统首次创建。

### 主作业 (Master Jobs)
通过直接启动可执行程序（应用/小程序）产生。

### 子作业 (Child Jobs)
由父作业生成，用于并行处理或子工作流。

## 执行作业

### 运行小程序

**基础执行**：
```python
import dxpy

# 运行小程序
job = dxpy.DXApplet("applet-xxxx").run({
    "input1": {"$dnanexus_link": "file-yyyy"},
    "input2": "parameter_value"
})

print(f"作业ID: {job.get_id()}")
```

**命令行方式**：
```bash
dx run applet-xxxx -i input1=file-yyyy -i input2="value"
```

### 运行应用

```python
# 按名称运行应用
job = dxpy.DXApp(name="my-app").run({
    "reads": {"$dnanexus_link": "file-xxxx"},
    "quality_threshold": 30
})
```

### 指定执行参数

```python
job = dxpy.DXApplet("applet-xxxx").run(
    applet_input={
        "input_file": {"$dnanexus_link": "file-yyyy"}
    },
    project="project-zzzz",  # 输出项目
    folder="/results",        # 输出目录
    name="我的分析作业",       # 作业名称
    instance_type="mem2_hdd2_x4",  # 覆盖实例类型
    priority="high"           # 作业优先级
)
```

## 作业监控

### 检查作业状态

```python
job = dxpy.DXJob("job-xxxx")
state = job.describe()["state"]

# 状态: idle, waiting_on_input, runnable, running, done, failed, terminated
print(f"作业状态: {state}")
```

**命令行方式**：
```bash
dx watch job-xxxx
```

### 等待作业完成

```python
# 阻塞直至作业完成
job.wait_on_done()

# 检查是否成功
if job.describe()["state"] == "done":
    output = job.describe()["output"]
    print(f"作业完成: {output}")
else:
    print("作业失败")
```

### 获取作业输出

```python
job = dxpy.DXJob("job-xxxx")

# 等待完成
job.wait_on_done()

# 获取输出
output = job.describe()["output"]
output_file_id = output["result_file"]["$dnanexus_link"]

# 下载结果
dxpy.download_dxfile(output_file_id, "result.txt")
```

### 作业输出引用

在作业完成前创建输出引用：

```python
# 启动第一个作业
job1 = dxpy.DXApplet("applet-1").run({"input": "..."})

# 使用输出引用启动第二个作业
job2 = dxpy.DXApplet("applet-2").run({
    "input": dxpy.dxlink(job1.get_output_ref("output_name"))
})
```

## 作业日志

### 查看日志

**命令行方式**：
```bash
dx watch job-xxxx --get-streams
```

**编程方式**：
```python
import sys

# 获取作业日志
job = dxpy.DXJob("job-xxxx")
log = dxpy.api.job_get_log(job.get_id())

for log_entry in log["loglines"]:
    print(log_entry)
```

## 并行执行

### 创建子作业

```python
@dxpy.entry_point('main')
def main(input_files):
    # 创建并行处理的子作业
    subjobs = []

    for input_file in input_files:
        subjob = dxpy.new_dxjob(
            fn_input={"file": input_file},
            fn_name="process_file"
        )
        subjobs.append(subjob)

    # 收集结果
    results = []
    for subjob in subjobs:
        result = subjob.get_output_ref("processed_file")
        results.append(result)

    return {"all_results": results}

@dxpy.entry_point('process_file')
def process_file(file):
    # 处理单个文件
    # ...
    return {"processed_file": output_file}
```

### 分散-收集模式

```python
# 分散: 并行处理条目
scatter_jobs = []
for item in items:
    job = dxpy.new_dxjob(
        fn_input={"item": item},
        fn_name="process_item"
    )
    scatter_jobs.append(job)

# 收集: 合并结果
gather_job = dxpy.new_dxjob(
    fn_input={
        "results": [job.get_output_ref("result") for job in scatter_jobs]
    },
    fn_name="combine_results"
)
```

## 工作流

工作流将多个应用/小程序组合成多步骤分析管道。

### 创建工作流

```python
# 创建工作流
workflow = dxpy.new_dxworkflow(
    name="我的分析管道",
    project="project-xxxx"
)

# 添加阶段
stage1 = workflow.add_stage(
    dxpy.DXApplet("applet-1"),
    name="质量控制",
    folder="/qc"
)

stage2 = workflow.add_stage(
    dxpy.DXApplet("applet-2"),
    name="序列比对",
    folder="/alignment"
)

# 连接阶段
stage2.set_input("reads", stage1.get_output_ref("filtered_reads"))

# 关闭工作流
workflow.close()
```

### 运行工作流

```python
# 运行工作流
analysis = workflow.run({
    "stage-xxxx.input1": {"$dnanexus_link": "file-yyyy"}
})

# 监控分析（作业集合）
analysis.wait_on_done()

# 获取工作流输出
outputs = analysis.describe()["output"]
```

**命令行方式**：
```bash
dx run workflow-xxxx -i stage-1.input=file-yyyy
```

## 作业权限与上下文

### 工作区上下文

作业在克隆输入数据的工作区项目中运行：
- 作业需要工作区的 `CONTRIBUTE` 权限
- 作业需要源项目的 `VIEW` 访问权限
- 所有费用计入发起项目

### 数据要求

作业启动前需满足：
1. 所有输入数据对象处于 `closed` 状态
2. 具备所需权限
3. 资源已分配

输出对象必须在工作区清理前达到 `closed` 状态。

## 作业生命周期

```
创建 → 等待输入 → 可运行 → 运行中 → 完成/失败
```

**状态说明**：
- `idle`: 作业已创建但未进入队列
- `waiting_on_input`: 等待输入数据对象关闭
- `runnable`: 准备就绪，等待资源分配
- `running`: 正在执行
- `done`: 成功完成
- `failed`: 执行失败
- `terminated`: 手动终止

## 错误处理

### 作业失败

```python
job = dxpy.DXJob("job-xxxx")
job.wait_on_done()

desc = job.describe()
if desc["state"] == "failed":
    print(f"作业失败: {desc.get('failureReason', '未知原因')}")
    print(f"失败信息: {desc.get('failureMessage', '')}")
```

### 重试失败作业

```python
# 重新运行失败作业
new_job = dxpy.DXApplet(desc["applet"]).run(
    desc["originalInput"],
    project=desc["project"]
)
```

### 终止作业

```python
# 停止运行中的作业
job = dxpy.DXJob("job-xxxx")
job.terminate()
```

**命令行方式**：
```bash
dx terminate job-xxxx
```

## 资源管理

### 实例类型

指定计算资源：

```python
# 使用特定实例类型运行
job = dxpy.DXApplet("applet-xxxx").run(
    {"input": "..."},
    instance_type="mem3_ssd1_v2_x8"  # 8核/高内存/SSD
)
```

常用实例类型：
- `mem1_ssd1_v2_x4` - 4核/标准内存
- `mem2_ssd1_v2_x8` - 8核/高内存
- `mem3_ssd1_v2_x16` - 16核/极高内存
- `mem1_ssd1_v2_x36` - 36核（并行工作负载）

### 超时设置

设置最大执行时间：

```python
job = dxpy.DXApplet("applet-xxxx").run(
    {"input": "..."},
    timeout="24h"  # 最长运行时间
)
```

## 作业标记与元数据

### 添加作业标签

```python
job = dxpy.DXApplet("applet-xxxx").run(
    {"input": "..."},
    tags=["实验1", "批次2", "生产环境"]
)
```

### 添加作业属性

```python
job = dxpy.DXApplet("applet-xxxx").run(
    {"input": "..."},
    properties={
        "experiment": "exp001",
        "sample": "样本1",
        "batch": "批次2"
    }
)
```

### 查找作业

```python
# 按标签查找作业
jobs = dxpy.find_jobs(
    project="project-xxxx",
    tags=["实验1"],
    describe=True
)

for job in jobs:
    print(f"{job['describe']['name']}: {job['id']}")
```

## 最佳实践

1. **作业命名**：使用描述性名称便于追踪
2. **标签与属性**：通过标记实现作业组织和检索
3. **资源选择**：根据工作负载匹配合适的实例类型
4. **错误处理**：检查作业状态并优雅处理失败
5. **并行处理**：对独立任务使用子作业并行
6. **工作流**：复杂多步骤分析使用工作流
7. **监控**：监控长时作业并检查日志
8. **成本管理**：平衡成本与性能选择实例类型
9. **超时设置**：设置合理超时防止失控作业
10. **清理**：移除失败或废弃作业

## 调试技巧

1. **检查日志**：始终查看作业日志中的错误信息
2. **验证输入**：确保输入文件已关闭且可访问
3. **本地测试**：部署前在本地验证逻辑
4. **小规模测试**：扩展前先用小数据集测试
5. **监控资源**：检查作业是否耗尽内存或磁盘
6. **实例类型**：资源不足时尝试更大实例
