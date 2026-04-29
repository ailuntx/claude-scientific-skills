# DNAnexus 数据操作

## 概述

DNAnexus 为文件、记录、数据库和其他数据对象提供全面的数据管理功能。所有数据操作均可通过 Python SDK (dxpy) 或命令行接口 (dx) 执行。

## 数据对象类型

### 文件
存储在平台上的二进制或文本数据。

### 记录
具有任意 JSON 详细信息和元数据的结构化数据对象。

### 数据库
用于关系数据的结构化数据库对象。

### 小程序和应用
可执行程序（在 app-development.md 中介绍）。

### 工作流
多步骤分析流程。

## 数据对象生命周期

### 状态

**开放状态**：数据可修改
- 文件：内容可写入
- 记录：详细信息可更新
- 小程序：默认在关闭状态下创建

**关闭状态**：数据变为不可变
- 文件内容固定
- 元数据字段被锁定（类型、详细信息、链接、可见性）
- 对象可共享并用于分析

### 状态转换

```
创建 (开放) → 修改 → 关闭 (不可变)
```

大多数对象以开放状态开始，需要显式关闭：
```python
# 关闭文件
file_obj.close()
```

部分对象可在一次操作中创建并关闭：
```python
# 创建已关闭的记录
record = dxpy.new_dxrecord(details={...}, close=True)
```

## 文件操作

### 上传文件

**从本地文件上传**：
```python
import dxpy

# 上传文件
file_obj = dxpy.upload_local_file("data.txt", project="project-xxxx")
print(f"已上传: {file_obj.get_id()}")
```

**带元数据上传**：
```python
file_obj = dxpy.upload_local_file(
    "data.txt",
    name="my_data",
    project="project-xxxx",
    folder="/results",
    properties={"sample": "sample1", "type": "raw"},
    tags=["experiment1", "batch2"]
)
```

**流式上传**：
```python
# 适用于大文件或生成的数据
file_obj = dxpy.new_dxfile(project="project-xxxx", name="output.txt")
file_obj.write("Line 1\n")
file_obj.write("Line 2\n")
file_obj.close()
```

### 下载文件

**下载到本地文件**：
```python
# 通过 ID 下载
dxpy.download_dxfile("file-xxxx", "local_output.txt")

# 使用句柄下载
file_obj = dxpy.DX
