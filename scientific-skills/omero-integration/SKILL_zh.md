```markdown
---
name: omero-integration
description: 显微影像数据管理平台。通过Python访问图像、检索数据集、分析像素、管理ROI/注释、批处理，适用于高通量筛选和显微工作流。
license: 未知
metadata:
    skill-author: K-Dense Inc.
---

# OMERO 集成

## 概述

OMERO 是一个用于管理、可视化和分析显微图像及元数据的开源平台。通过 Python API 访问图像、检索数据集、分析像素、管理 ROI 和注释，适用于高通量筛选和显微工作流。

## 使用场景

本技能适用于以下场景：
- 使用 OMERO Python API (omero-py) 访问显微数据
- 以编程方式检索图像、数据集、项目或筛选数据
- 分析像素数据并创建衍生图像
- 在显微图像上创建或管理 ROI（感兴趣区域）
- 为 OMERO 对象添加注释、标签或元数据
- 在 OMERO 表格中存储测量结果
- 创建服务端脚本进行批处理
- 执行高通量筛选分析

## 核心能力

本技能涵盖八大核心能力领域，详细文档见 references/ 目录：

### 1. 连接与会话管理
**文件**：`references/connection.md`

建立到 OMERO 服务器的安全连接，管理会话，处理身份验证，并操作组上下文。用于初始设置和连接模式。

**常见场景：**
- 使用凭证连接 OMERO 服务器
- 复用现有会话 ID
- 切换组上下文
- 使用上下文管理器管理连接生命周期

### 2. 数据访问与检索
**文件**：`references/data_access.md`

导航 OMERO 层级数据结构（项目→数据集→图像）和筛选数据（筛选板→孔板→孔位）。检索对象、按属性查询并访问元数据。

**常见场景：**
- 列出用户的所有项目和数据集
- 按 ID 或数据集检索图像
- 访问筛选板数据
- 使用过滤器查询对象

### 3. 元数据与注释
**文件**：`references/metadata.md`

创建和管理注释，包括标签、键值对、文件附件和评论。将注释关联到图像、数据集或其他对象。

**常见场景：**
- 为图像添加标签
- 将分析结果作为文件附件
- 创建自定义键值元数据
- 按命名空间查询注释

### 4. 图像处理与渲染
**文件**：`references/image_processing.md`

以 NumPy 数组形式访问原始像素数据，操作渲染设置，创建衍生图像，并管理物理维度。

**常见场景：**
- 提取像素数据进行计算分析
- 生成缩略图
- 创建最大强度投影
- 修改通道渲染设置

### 5. 感兴趣区域 (ROI)
**文件**：`references/rois.md`

创建、检索和分析多种形状的 ROI（矩形、椭圆、多边形、掩膜、点、线）。提取 ROI 区域的强度统计值。

**常见场景：**
- 在图像上绘制矩形 ROI
- 创建多边形掩膜进行分割
- 分析 ROI 内的像素强度
- 导出 ROI 坐标

### 6. OMERO 表格
**文件**：`references/tables.md`

存储和查询与 OMERO 对象关联的结构化表格数据。适用于分析结果、测量数据和元数据。

**常见场景：**
- 存储图像的量化测量结果
- 创建多列类型的表格
- 按条件查询表格数据
- 将表格关联到特定图像或数据集

### 7. 脚本与批处理
**文件**：`references/scripts.md`

创建在服务端运行的 OMERO.scripts，用于批处理、自动化工作流以及与 OMERO 客户端集成。

**常见场景：**
- 批量处理多张图像
- 创建自动化分析流水线
- 跨数据集生成统计摘要
- 以自定义格式导出数据

### 8. 高级功能
**文件**：`references/advanced.md`

涵盖权限管理、文件集、跨组查询、删除操作及其他高级功能。

**常见场景：**
- 处理组权限
- 访问原始导入文件
- 执行跨组查询
- 使用回调删除对象

## 安装

```bash
uv pip install omero-py
```

**要求：**
- Python 3.7+
- Zeroc Ice 3.6+
- 可访问的 OMERO 服务器（主机、端口、凭证）

## 快速入门

基础连接模式：

```python
from omero.gateway import BlitzGateway

# 连接 OMERO 服务器
conn = BlitzGateway(username, password, host=host, port=port)
connected = conn.connect()

if connected:
    # 执行操作
    for project in conn.listProjects():
        print(project.getName())

    # 始终关闭连接
    conn.close()
else:
    print("连接失败")
```

**推荐使用上下文管理器模式：**

```python
from omero.gateway import BlitzGateway

with BlitzGateway(username, password, host=host, port=port) as conn:
    # 连接自动管理
    for project in conn.listProjects():
        print(project.getName())
    # 退出时自动关闭
```

## 能力选择指南

**数据探索场景：**
- 通过 `references/connection.md` 建立连接
- 使用 `references/data_access.md` 导航层级
- 查看 `references/metadata.md` 获取注释详情

**图像分析场景：**
- 使用 `references/image_processing.md` 访问像素数据
- 使用 `references/rois.md` 进行区域分析
- 使用 `references/tables.md` 存储结果

**自动化场景：**
- 使用 `references/scripts.md` 进行服务端处理
- 使用 `references/data_access.md` 批量检索数据

**高级操作场景：**
- 使用 `references/advanced.md` 处理权限和删除
- 查看 `references/connection.md` 实现跨组查询

## 典型工作流

### 工作流 1：检索与分析图像

1. 连接 OMERO 服务器 (`references/connection.md`)
2. 导航至数据集 (`references/data_access.md`)
3. 从数据集检索图像 (`references/data_access.md`)
4. 以 NumPy 数组形式访问像素数据 (`references/image_processing.md`)
5. 执行分析
6. 将结果存储为表格或文件注释 (`references/tables.md` 或 `references/metadata.md`)

### 工作流 2：批量 ROI 分析

1. 连接 OMERO 服务器
2. 检索带现有 ROI 的图像 (`references/rois.md`)
3. 对每张图像获取 ROI 形状
4. 提取 ROI 内像素强度 (`references/rois.md`)
5. 将测量值存入 OMERO 表格 (`references/tables.md`)

### 工作流 3：创建分析脚本

1. 设计分析流程
2. 使用 OMERO.scripts 框架 (`references/scripts.md`)
3. 通过脚本参数访问数据
4. 批量处理图像
5. 生成输出（新图像、表格、文件）

## 错误处理

始终使用 try-except 块包裹 OMERO 操作，并确保连接正确关闭：

```python
from omero.gateway import BlitzGateway
import traceback

try:
    conn = BlitzGateway(username, password, host=host, port=port)
    if not conn.connect():
        raise Exception("连接失败")

    # 执行操作

except Exception as e:
    print(f"错误: {e}")
    traceback.print_exc()
finally:
    if conn:
        conn.close()
```

## 扩展资源

- **官方文档**: https://omero.readthedocs.io/en/stable/developers/Python.html
- **BlitzGateway API**: https://omero.readthedocs.io/en/stable/developers/Python.html#omero-blitzgateway
- **OMERO 数据模型**: https://omero.readthedocs.io/en/stable/developers/Model.html
- **社区论坛**: https://forum.image.sc/tag/omero

## 注意事项

- OMERO 采用基于组的权限控制（只读、注释、读写）
- 图像组织结构：项目 > 数据集 > 图像
- 筛选数据结构：筛选板 > 孔板 > 孔位 > 孔位样本 > 图像
- 始终关闭连接以释放服务器资源
- 使用上下文管理器实现自动资源管理
- 像素数据以 NumPy 数组形式返回供分析使用
```
