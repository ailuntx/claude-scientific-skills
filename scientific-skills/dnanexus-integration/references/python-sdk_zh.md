# DNAnexus Python SDK (dxpy)

## 概述

dxpy 库提供与 DNAnexus 平台交互的 Python 绑定。它既可在 DNAnexus 执行环境（用于平台上运行的应用程序）中使用，也可供访问 API 的外部脚本使用。

## 安装

```bash
# 安装 dxpy
pip install dxpy

# 或使用 conda
conda install -c bioconda dxpy
```

**要求**：Python 3.8 或更高版本

## 认证

### 登录

```bash
# 通过命令行登录
dx login
```

### API 令牌

```python
import dxpy

# 设置认证令牌
dxpy.set_security_context({
    "auth_token_type": "Bearer",
    "auth_token": "YOUR_API_TOKEN"
})
```

### 环境变量

```bash
# 通过环境变量设置令牌
export DX_SECURITY_CONTEXT='{"auth_token": "YOUR_TOKEN", "auth_token_type": "Bearer"}'
```

## 核心类

### DXFile

文件对象的处理程序。

```python
import dxpy

# 获取文件处理程序
file_obj = dxpy.DXFile("file-xxxx")

# 获取文件信息
desc = file_obj.describe()
print(f"名称: {desc['name']}")
print(f"大小: {desc['size']} 字节")

# 下载文件
dxpy.download_dxfile(file_obj.get_id(), "local_file.txt")

# 读取文件内容
with file_obj.open_file() as f:
    contents = f.read()

# 更新元数据
file_obj.set_properties({"key": "value"})
file_obj.add_tags(["tag1", "tag2"])
file_obj.rename("new_name.txt")

# 关闭文件
file_obj.close()
```

### DXRecord

记录对象的处理程序。

```python
# 创建记录
record = dxpy.new_dxrecord(
    name="metadata",
    types=["Metadata"],
    details={"key": "value"},
    project="project-xxxx",
    close=True
)

# 获取记录处理程序
record = dxpy.DXRecord("record-xxxx")

# 获取详情
details = record.get_details()

# 更新详情（必须处于开启状态）
record.set_details({"updated": True})
record.close()
```

### DXApplet

小程序对象的处理程序。

```python
# 获取小程序
applet = dxpy.DXApplet("applet-xxxx")

# 获取小程序信息
desc = applet.describe()
print(f"名称: {desc['name']}")
print(f"版本: {desc.get('version', 'N/A')}")

# 运行小程序
job = applet.run({
    "input1": {"$dnanexus_link": "file-yyyy"},
    "param1": "value"
})
```

### DXApp

应用对象的处理程序。

```python
# 按名称获取应用
app = dxpy.DXApp(name="my-app")

# 或按 ID 获取
app = dxpy.DXApp("app-xxxx")

# 运行应用
job = app.run({
    "input": {"$dnanexus_link": "file-yyyy"}
})
```

### DXWorkflow

工作流对象的处理程序。

```python
# 创建工作流
workflow = dxpy.new_dxworkflow(
    name="我的流程",
    project="project-xxxx"
)

# 添加阶段
stage = workflow.add_stage(
    dxpy.DXApplet("applet-xxxx"),
    name="步骤 1"
)

# 设置阶段输入
stage.set_input("input1", {"$dnanexus_link": "file-yyyy"})

# 关闭工作流
workflow.close()

# 运行工作流
analysis = workflow.run({})
```

### DXJob

作业对象的处理程序。

```python
# 获取作业
job = dxpy.DXJob("job-xxxx")

# 获取作业信息
desc = job.describe()
print(f"状态: {desc['state']}")
print(f"名称: {desc['name']}")

# 等待完成
job.wait_on_done()

# 获取输出
output = desc.get("output", {})

# 终止作业
job.terminate()
```

### DXProject

项目对象的处理程序。

```python
# 获取项目
project = dxpy.DXProject("project-xxxx")

# 获取项目信息
desc = project.describe()
print(f"名称: {desc['name']}")
print(f"区域: {desc.get('region', 'N/A')}")

# 列出文件夹内容
contents = project.list_folder("/data")
print(f"对象: {contents['objects']}")
print(f"文件夹: {contents['folders']}")
```

## 高级函数

### 文件操作

```python
# 上传文件
file_obj = dxpy.upload_local_file(
    "local_file.txt",
    project="project-xxxx",
    folder="/data",
    name="uploaded_file.txt"
)

# 下载文件
dxpy.download_dxfile("file-xxxx", "downloaded.txt")

# 将字符串作为文件上传
file_obj = dxpy.upload_string("Hello World", project="project-xxxx")
```

### 创建数据对象

```python
# 新建文件
file_obj = dxpy.new_dxfile(
    project="project-xxxx",
    name="output.txt"
)
file_obj.write("内容")
file_obj.close()

# 新建记录
record = dxpy.new_dxrecord(
    name="metadata",
    details={"key": "value"},
    project="project-xxxx"
)
```

### 搜索函数

```python
# 查找数据对象
results = dxpy.find_data_objects(
    classname="file",
    name="*.fastq",
    project="project-xxxx",
    folder="/raw_data",
    describe=True
)

for result in results:
    print(f"{result['describe']['name']}: {result['id']}")

# 查找项目
projects = dxpy.find_projects(
    name="*analysis*",
    describe=True
)

# 查找作业
jobs = dxpy.find_jobs(
    project="project-xxxx",
    created_after="2025-01-01",
    state="failed"
)

# 查找应用
apps = dxpy.find_apps(
    category="Read Mapping"
)
```

### 链接与引用

```python
# 创建数据对象链接
link = dxpy.dxlink("file-xxxx")
# 返回: {"$dnanexus_link": "file-xxxx"}

# 创建带项目的链接
link = dxpy.dxlink("file-xxxx", "project-yyyy")

# 获取作业输出引用（用于作业链）
output_ref = job.get_output_ref("output_name")
```

## API 方法

### 直接 API 调用

针对高级函数未覆盖的操作：

```python
# 直接调用 API 方法
result = dxpy.api.project_new({
    "name": "新项目",
    "description": "通过 API 创建"
})

project_id = result["id"]

# 文件描述
file_desc = dxpy.api.file_describe("file-xxxx")

# 系统查找数据对象
results = dxpy.api.system_find_data_objects({
    "class": "file",
    "project": "project-xxxx",
    "name": {"regexp": ".*\\.bam$"}
})
```

### 常用 API 方法

```python
# 项目操作
dxpy.api.project_invite("project-xxxx", {"invitee": "user-yyyy", "level": "VIEW"})
dxpy.api.project_new_folder("project-xxxx", {"folder": "/new_folder"})

# 文件操作
dxpy.api.file_close("file-xxxx")
dxpy.api.file_remove("file-xxxx")

# 作业操作
dxpy.api.job_terminate("job-xxxx")
dxpy.api.job_get_log("job-xxxx")
```

## 应用开发函数

### 入口点

```python
import dxpy

@dxpy.entry_point('main')
def main(input1, input2):
    """应用主入口点"""
    # 处理输入
    result = process(input1, input2)

    # 返回输出
    return {
        "output1": result
    }

# 应用代码末尾必需
dxpy.run()
```

### 创建子作业

```python
# 在应用中生成子作业
subjob = dxpy.new_dxjob(
    fn_input={"input": value},
    fn_name="helper_function"
)

# 获取输出引用
output_ref = subjob.get_output_ref("result")

@dxpy.entry_point('helper_function')
def helper_function(input):
    # 处理过程
    return {"result": output}
```

## 错误处理

### 异常类型

```python
import dxpy
from dxpy.exceptions import DXError, DXAPIError

try:
    file_obj = dxpy.DXFile("file-xxxx")
    desc = file_obj.describe()
except DXAPIError as e:
    print(f"API 错误: {e}")
    print(f"状态码: {e.code}")
except DXError as e:
    print(f"通用错误: {e}")
```

### 常见异常

- `DXAPIError`: API 请求失败
- `DXError`: 通用 DNAnexus 错误
- `ResourceNotFound`: 对象不存在
- `PermissionDenied`: 权限不足
- `InvalidInput`: 无效输入参数

## 工具函数

### 获取处理程序

```python
# 通过 ID/链接获取处理程序
handler = dxpy.get_handler("file-xxxx")
# 根据对象类型返回 DXFile、DXRecord 等

# 将处理程序绑定到项目
handler = dxpy.DXFile("file-xxxx", project="project-yyyy")
```

### 描述方法

```python
# 描述任意对象
desc = dxpy.describe("file-xxxx")
print(desc)

# 指定字段描述
desc = dxpy.describe("file-xxxx", fields={"name": True, "size": True})
```

## 配置

### 设置项目上下文

```python
# 设置默认项目
dxpy.set_workspace_id("project-xxxx")

# 获取当前项目
project_id = dxpy.WORKSPACE_ID
```

### 设置区域

```python
# 设置 API 服务器
dxpy.set_api_server_info(host="api.dnanexus.com", port=443)
```

## 最佳实践

1. **使用高级函数**：优先使用 `upload_local_file()` 而非手动创建文件
2. **处理程序复用**：创建处理程序后应重复使用
3. **批量操作**：使用查找函数处理多个对象
4. **错误处理**：始终用 try-except 块包装 API 调用
5. **关闭对象**：修改后记得关闭文件和记录
6. **项目上下文**：为应用设置工作区上下文
7. **API 令牌安全**：切勿在源代码中硬编码令牌
8. **描述字段**：仅请求必要字段以减少延迟
9. **搜索过滤**：使用具体过滤条件缩小搜索结果范围
10. **链接格式**：使用 `dxpy.dxlink()` 确保链接创建一致性

## 常用模式

### 上传与处理模式

```python
# 上传输入
input_file = dxpy.upload_local_file("data.txt", project="project-xxxx")

# 运行分析
job = dxpy.DXApplet("applet-xxxx").run({
    "input": dxpy.dxlink(input_file.get_id())
})

# 等待并下载结果
job.wait_on_done()
output_id = job.describe()["output"]["result"]["$dnanexus_link"]
dxpy.download_dxfile(output_id, "result.txt")
```

### 批量文件处理

```python
# 查找所有 FASTQ 文件
files = dxpy.find_data_objects(
    classname="file",
    name="*.fastq",
    project="project-xxxx"
)

# 处理每个文件
jobs = []
for file_result in files:
    job = dxpy.DXApplet("applet-xxxx").run({
        "input": dxpy.dxlink(file_result["id"])
    })
    jobs.append(job)

# 等待所有作业
for job in jobs:
    job.wait_on_done()
    print(f"作业 {job.get_id()} 已完成")
```

### 带依赖的工作流

```python
# 作业 1
job1 = applet1.run({"input": data})

# 作业 2 依赖作业 1 的输出
job2 = applet2.run({
    "input": job1.get_output_ref("result")
})

# 作业 3 依赖作业 2
job3 = applet3.run({
    "input": job2.get_output_ref("processed")
})

# 等待最终结果
job3.wait_on_done()
```
