# DNAnexus 应用开发

## 概述

应用（Apps）和应用小程序（Applets）是在 DNAnexus 平台上运行的可执行程序。它们可以用 Python 或 Bash 编写，并随所有必要的依赖项和配置一起部署。

## 应用小程序 vs 应用

- **应用小程序**：存在于项目内部的数据对象。适用于开发和测试。
- **应用**：版本化、可共享的可执行文件，不存储在项目内。可发布供他人使用。

两者在最终构建步骤前的创建方式相同。应用小程序后续可转换为应用。

## 创建应用/应用小程序

### 使用 dx-app-wizard

生成应用骨架目录结构：

```bash
dx-app-wizard
```

这将创建：
- `dxapp.json` - 配置文件
- `src/` - 源代码目录
- `resources/` - 打包的依赖项
- `test/` - 测试文件

### 构建与部署

构建应用小程序：
```bash
dx build
```

构建应用：
```bash
dx build --app
```

构建流程：
1. 验证 dxapp.json 配置
2. 打包源代码和资源
3. 部署到平台
4. 返回应用小程序/应用 ID

## 应用目录结构

```
my-app/
├── dxapp.json          # 元数据和配置
├── src/
│   └── my-app.py       # 主可执行文件 (Python)
│   └── my-app.sh       # 或 Bash 脚本
├── resources/          # 打包的文件和依赖项
│   └── tools/
│   └── data/
└── test/               # 测试数据和脚本
    └── test.json
```

## Python 应用结构

### 入口点

Python 应用使用 `@dxpy.entry_point()` 装饰器定义函数：

```python
import dxpy

@dxpy.entry_point('main')
def main(input1, input2):
    # 处理输入
    # 返回输出
    return {
        "output1": result1,
        "output2": result2
    }

dxpy.run()
```

### 输入/输出处理

**输入**：DNAnexus 数据对象表示为包含链接的字典：

```python
@dxpy.entry_point('main')
def main(reads_file):
    # 将链接转换为句柄
    reads_dxfile = dxpy.DXFile(reads_file)

    # 下载到本地文件系统
    dxpy.download_dxfile(reads_dxfile.get_id(), "reads.fastq")

    # 处理文件...
```

**输出**：直接返回基本类型，将文件输出转换为链接：

```python
    # 上传结果文件
    output_file = dxpy.upload_local_file("output.fastq")

    return {
        "trimmed_reads": dxpy.dxlink(output_file)
    }
```

## Bash 应用结构

Bash 应用采用更简单的 shell 脚本方式：

```bash
#!/bin/bash
set -e -x -o pipefail

main() {
    # 下载输入
    dx download "$reads_file" -o reads.fastq

    # 处理
    process_reads reads.fastq > output.fastq

    # 上传输出
    trimmed_reads=$(dx upload output.fastq --brief)

    # 设置任务输出
    dx-jobutil-add-output trimmed_reads "$trimmed_reads" --class=file
}
```

## 常见开发模式

### 1. 生物信息学流程

下载 → 处理 → 上传模式：

```python
# 下载输入
dxpy.download_dxfile(input_file_id, "input.fastq")

# 运行分析
subprocess.check_call(["tool", "input.fastq", "output.bam"])

# 上传结果
output = dxpy.upload_local_file("output.bam")
return {"aligned_reads": dxpy.dxlink(output)}
```

### 2. 多文件处理

```python
# 处理多个输入
for file_link in input_files:
    file_handler = dxpy.DXFile(file_link)
    local_path = f"{file_handler.name}"
    dxpy.download_dxfile(file_handler.get_id(), local_path)
    # 处理每个文件...
```

### 3. 并行处理

应用可创建子任务实现并行执行：

```python
# 创建子任务
subjobs = []
for item in input_list:
    subjob = dxpy.new_dxjob(
        fn_input={"input": item},
        fn_name="process_item"
    )
    subjobs.append(subjob)

# 收集结果
results = [job.get_output_ref("result") for job in subjobs]
```

## 执行环境

应用在隔离的 Linux 虚拟机 (Ubuntu 24.04) 中运行，具备：
- 互联网访问权限
- DNAnexus API 访问权限
- `/home/dnanexus` 临时存储空间
- 输入文件下载到任务工作区
- 安装依赖项的 root 权限

## 测试应用

### 本地测试

部署前本地测试应用逻辑：

```bash
cd my-app
python src/my-app.py
```

### 平台测试

在平台上运行应用小程序：

```bash
dx run applet-xxxx -i input1=file-yyyy
```

监控任务执行：

```bash
dx watch job-zzzz
```

查看任务日志：

```bash
dx watch job-zzzz --get-streams
```

## 最佳实践

1. **错误处理**：使用 try-except 块并提供明确的错误信息
2. **日志记录**：将进度和调试信息输出到 stdout/stderr
3. **输入验证**：处理前验证输入
4. **清理**：完成后删除临时文件
5. **文档**：在 dxapp.json 中包含清晰描述
6. **测试**：使用多种输入类型和边界情况测试
7. **版本控制**：为应用使用语义化版本

## 常见问题

### 文件未找到
确保访问前文件已正确下载：
```python
dxpy.download_dxfile(file_id, local_path)
# 现在可安全打开 local_path
```

### 内存不足
在 dxapp.json 的 systemRequirements 中指定更大的实例类型

### 超时
在 dxapp.json 中增加超时时间或拆分为更小的任务

### 权限错误
确保应用在 dxapp.json 中具有必要权限
