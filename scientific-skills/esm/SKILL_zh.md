---
name: esm
description: 用于蛋白质语言模型的综合工具包，包含ESM3（跨序列、结构和功能的生成式多模态蛋白质设计）和ESM C（高效蛋白质嵌入与表征）。适用于处理蛋白质序列、结构或功能预测；设计新型蛋白质；生成蛋白质嵌入；执行反向折叠；或开展蛋白质工程任务。支持本地模型使用和基于云的Forge API，实现可扩展推理。
license: MIT许可证
metadata:
    skill-author: K-Dense Inc.
---

# ESM：进化尺度建模

## 概述

ESM提供最先进的蛋白质语言模型，用于理解、生成和设计蛋白质。本工具支持两大模型家族：ESM3用于跨序列、结构和功能的生成式蛋白质设计，ESM C用于高效蛋白质表征学习与嵌入。

## 核心功能

### 1. ESM3蛋白质序列生成

通过多模态生成模型创建具有特定属性的新型蛋白质序列。

**适用场景：**
- 设计具备特定功能特性的蛋白质
- 补全部分蛋白质序列
- 生成现有蛋白质变体
- 创建具有目标结构特征的蛋白质

**基础用法：**

```python
from esm.models.esm3 import ESM3
from esm.sdk.api import ESM3InferenceClient, ESMProtein, GenerationConfig

# 本地加载模型
model: ESM3InferenceClient = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")

# 创建蛋白质提示
protein = ESMProtein(sequence="MPRT___KEND")  # '_'代表掩码位置

# 生成补全序列
protein = model.generate(protein, GenerationConfig(track="sequence", num_steps=8))
print(protein.sequence)
```

**通过Forge API远程/云端使用：**

```python
from esm.sdk.forge import ESM3ForgeInferenceClient
from esm.sdk.api import ESMProtein, GenerationConfig

# 连接Forge
model = ESM3ForgeInferenceClient(model="esm3-medium-2024-08", url="https://forge.evolutionaryscale.ai", token="<token>")

# 生成序列
protein = model.generate(protein, GenerationConfig(track="sequence", num_steps=8))
```

详细ESM3模型规格、高级生成配置及多模态提示示例见`references/esm3-api.md`。

### 2. 结构预测与反向折叠

使用ESM3结构轨道实现序列到结构的预测或反向折叠（根据结构设计序列）。

**结构预测：**

```python
from esm.sdk.api import ESM3InferenceClient, ESMProtein, GenerationConfig

# 根据序列预测结构
protein = ESMProtein(sequence="MPRTKEINDAGLIVHSP...")
protein_with_structure = model.generate(
    protein,
    GenerationConfig(track="structure", num_steps=protein.sequence.count("_"))
)

# 获取预测结构
coordinates = protein_with_structure.coordinates  # 三维坐标
pdb_string = protein_with_structure.to_pdb()
```

**反向折叠（根据结构设计序列）：**

```python
# 为目标结构设计序列
protein_with_structure = ESMProtein.from_pdb("target_structure.pdb")
protein_with_structure.sequence = None  # 移除序列

# 生成折叠为此结构的序列
designed_protein = model.generate(
    protein_with_structure,
    GenerationConfig(track="sequence", num_steps=50, temperature=0.7)
)
```

### 3. ESM C蛋白质嵌入

生成高质量嵌入，用于功能预测、分类或相似性分析等下游任务。

**适用场景：**
- 提取蛋白质表征用于机器学习
- 计算序列相似性
- 蛋白质分类的特征提取
- 蛋白质相关任务的迁移学习

**基础用法：**

```python
from esm.models.esmc import ESMC
from esm.sdk.api import ESMProtein

# 加载ESM C模型
model = ESMC.from_pretrained("esmc-300m").to("cuda")

# 获取嵌入
protein = ESMProtein(sequence="MPRTKEINDAGLIVHSP...")
protein_tensor = model.encode(protein)

# 生成嵌入
embeddings = model.forward(protein_tensor)
```

**批量处理：**

```python
# 编码多个蛋白质
proteins = [
    ESMProtein(sequence="MPRTKEIND..."),
    ESMProtein(sequence="AGLIVHSPQ..."),
    ESMProtein(sequence="KTEFLNDGR...")
]

embeddings_list = [model.logits(model.forward(model.encode(p))) for p in proteins]
```

ESM C模型详情、效率比较和高级嵌入策略见`references/esm-c-api.md`。

### 4. 功能条件化与注释

使用ESM3功能轨道生成具有特定功能注释的蛋白质，或根据序列预测功能。

**功能条件化生成：**

```python
from esm.sdk.api import ESMProtein, FunctionAnnotation, GenerationConfig

# 创建含目标功能的蛋白质
protein = ESMProtein(
    sequence="_" * 200,  # 生成200个残基的蛋白质
    function_annotations=[
        FunctionAnnotation(label="fluorescent_protein", start=50, end=150)
    ]
)

# 生成具备指定功能的序列
functional_protein = model.generate(
    protein,
    GenerationConfig(track="sequence", num_steps=200)
)
```

### 5. 思维链生成

通过ESM3的思维链生成方法迭代优化蛋白质设计。

```python
from esm.sdk.api import GenerationConfig

# 多步优化
protein = ESMProtein(sequence="MPRT" + "_" * 100 + "KEND")

# 步骤1：生成初始结构
config = GenerationConfig(track="structure", num_steps=50)
protein = model.generate(protein, config)

# 步骤2：基于结构优化序列
config = GenerationConfig(track="sequence", num_steps=50, temperature=0.5)
protein = model.generate(protein, config)

# 步骤3：预测功能
config = GenerationConfig(track="function", num_steps=20)
protein = model.generate(protein, config)
```

### 6. Forge API批量处理

使用Forge异步执行器高效处理多个蛋白质。

```python
from esm.sdk.forge import ESM3ForgeInferenceClient
import asyncio

client = ESM3ForgeInferenceClient(model="esm3-medium-2024-08", token="<token>")

# 异步批量处理
async def batch_generate(proteins_list):
    tasks = [
        client.async_generate(protein, GenerationConfig(track="sequence"))
        for protein in proteins_list
    ]
    return await asyncio.gather(*tasks)

# 执行
proteins = [ESMProtein(sequence=f"MPRT{'_' * 50}KEND") for _ in range(10)]
results = asyncio.run(batch_generate(proteins))
```

详细Forge API文档、认证、速率限制和批处理模式见`references/forge-api.md`。

## 模型选择指南

**ESM3模型（生成式）：**
- `esm3-sm-open-v1` (1.4B) - 开放权重，本地使用，适合实验
- `esm3-medium-2024-08` (7B) - 最佳质量与速度平衡（仅限Forge）
- `esm3-large-2024-03` (98B) - 最高质量，速度较慢（仅限Forge）

**ESM C模型（嵌入）：**
- `esmc-300m` (30层) - 轻量级，快速推理
- `esmc-600m` (36层) - 均衡性能
- `esmc-6b` (80层) - 最高表征质量

**选择标准：**
- **本地开发/测试：** 使用`esm3-sm-open-v1`或`esmc-300m`
- **生产环境：** 通过Forge使用`esm3-medium-2024-08`
- **最高精度：** 使用`esm3-large-2024-03`或`esmc-6b`
- **高吞吐量：** 使用Forge API批量执行器
- **成本优化：** 使用小型模型，实施缓存策略

## 安装指南

**基础安装：**

```bash
uv pip install esm
```

**安装Flash Attention（推荐加速推理）：**

```bash
uv pip install esm
uv pip install flash-attn --no-build-isolation
```

**获取Forge API访问权限：**

```bash
uv pip install esm  # SDK包含Forge客户端
```

无需额外依赖。Forge API令牌请访问 https://forge.evolutionaryscale.ai

## 典型工作流

完整示例和工作流见`references/workflows.md`，包含：
- 基于思维链的新型GFP设计
- 蛋白质变体生成与筛选
- 基于结构的序列优化
- 功能预测流程
- 嵌入驱动的聚类分析

## 参考文档

本工具包含完整参考文档：
- `references/esm3-api.md` - ESM3架构、API参考、生成参数和多模态提示
- `references/esm-c-api.md` - ESM C模型详情、嵌入策略和性能优化
- `references/forge-api.md` - Forge平台文档、认证、批处理和部署
- `references/workflows.md` - 完整示例和典型工作流模式

这些文档包含详细API规范、参数说明和高级用法，可按需加载。

## 最佳实践

**生成任务：**
- 原型设计使用小型模型（`esm3-sm-open-v1`）
- 用temperature参数控制多样性（0.0=确定性，1.0=多样性）
- 复杂设计采用思维链迭代优化
- 通过结构预测或湿实验验证生成序列

**嵌入任务：**
- 尽可能批量处理序列提升效率
- 重复分析时缓存嵌入结果
- 计算相似性时标准化嵌入
- 根据下游任务需求选择合适模型尺寸

**生产部署：**
- 使用Forge API实现可扩展性
- 实施API调用的错误处理和重试逻辑
- 监控令牌用量并设置速率限制
- 专用基础设施建议部署AWS SageMaker

## 资源与文档

- **GitHub仓库：** https://github.com/evolutionaryscale/esm
- **Forge平台：** https://forge.evolutionaryscale.ai
- **科学论文：** Hayes et al., Science (2025) - https://www.science.org/doi/10.1126/science.ads0018
- **博客文章：**
  - ESM3发布：https://www.evolutionaryscale.ai/blog/esm3-release
  - ESM C发布：https://www.evolutionaryscale.ai/blog/esm-cambrian
- **社区：** Slack社区 https://bit.ly/3FKwcWd
- **模型权重：** HuggingFace EvolutionaryScale组织

## 负责任使用

ESM专为蛋白质工程、药物发现和科学研究的良性应用而设计。设计新型蛋白质时请遵循负责任生物设计框架（https://responsiblebiodesign.ai/）。实验验证前请评估蛋白质设计的生物安全与伦理影响。
