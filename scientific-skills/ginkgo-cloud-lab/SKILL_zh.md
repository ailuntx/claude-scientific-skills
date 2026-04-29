---
name: ginkgo-cloud-lab
description: 在Ginkgo Bioworks云实验室（cloud.ginkgo.bio）上提交和管理实验方案。这是一个基于Web的界面，用于在可重构自动化推车（RACs）上执行自主实验室操作。当用户需要运行无细胞蛋白表达（验证或优化）、生成荧光像素画或使用Ginkgo云实验室服务时使用。涵盖方案选择、输入准备、定价和订购流程。
---

# Ginkgo云实验室

## 概述

Ginkgo云实验室（https://cloud.ginkgo.bio）提供对Ginkgo Bioworks自主实验室基础设施的远程访问。实验方案在可重构自动化推车（RACs）上执行——这些模块化单元配备机械臂、磁悬浮样品传输系统，以及覆盖70多种仪器的工业级软件。

该平台还包含**EstiMate**人工智能代理，可接受人类语言描述的实验方案，并为超出预设方案的自定义流程提供可行性评估和定价。

## 可用方案

### 1. 无细胞蛋白表达验证
使用重构大肠杆菌CFPS系统进行快速表达筛选。提交FASTA序列（最长1800 bp）后，可获得表达确认、基础滴度（mg/L）、初始纯度及虚拟凝胶图像。
- **价格：** 39美元/样本 | **周期：** 5-10天 | **状态：** 认证
- **详情：** 参见[references/cell-free-protein-expression-validation.md](references/cell-free-protein-expression-validation.md)

### 2. 无细胞蛋白表达优化
基于实验设计（DoE）的优化方案，每个蛋白最多支持24种条件（裂解液、温度、分子伴侣、二硫键增强剂、辅因子）。专为难表达蛋白和膜蛋白设计。
- **价格：** 199美元/样本 | **周期：** 6-11天 | **状态：** 认证
- **详情：** 参见[references/cell-free-protein-expression-optimization.md](references/cell-free-protein-expression-optimization.md)

### 3. 荧光像素画生成
将像素画图像（48x48至96x96像素，PNG/SVG格式）通过声波点样技术转化为最多使用11种大肠杆菌菌株的荧光细菌艺术品，最终交付高分辨率紫外光照片。
- **价格：** 25美元/板 | **周期：** 5-7天 | **状态：** 测试版
- **详情：** 参见[references/fluorescent-pixel-art-generation.md](references/fluorescent-pixel-art-generation.md)

## 通用订购流程
1. 在https://cloud.ginkgo.bio/protocols选择方案
2. 配置参数（样本/蛋白数量、重复次数、培养板规格）
3. 上传输入文件（蛋白方案需FASTA文件，像素画需PNG/SVG文件）
4. 在"附加说明"字段添加特殊要求
5. 提交后接收可行性报告和报价单

对于未列出的方案，使用**EstiMate**聊天功能以自然语言描述自定义方案，获取兼容性评估和定价。

## 认证访问
访问Ginkgo云实验室：https://cloud.ginkgo.bio。可能需要创建账户或机构授权。访问问题请联系cloud@ginkgo.bio。

## 核心基础设施
- **RACs（可重构自动化推车）：** 配备高精度机械臂和磁悬浮传输系统的模块化机器人单元
- **Catalyst软件：** 方案编排、调度、参数化和实时监控系统
- **70+集成仪器：** 样品制备、液体处理、分析读数、存储和培养设备
- **Nebula：** Ginkgo位于马萨诸塞州波士顿的自主实验室设施
