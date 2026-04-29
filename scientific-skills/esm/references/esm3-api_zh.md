# ESM3 API 参考

## 概述

ESM3 是一个前沿的多模态生成语言模型，能够对蛋白质的序列、结构和功能进行联合推理。该模型采用迭代掩码语言建模技术，实现跨三种模态的同步生成。

## 模型架构

**ESM3 系列模型：**

| 模型 ID | 参数量 | 可用性 | 最佳适用场景 |
|----------|-----------|--------------|----------|
| `esm3-sm-open-v1` | 14亿 | 开放权重（本地） | 开发、测试、学习 |
| `esm3-medium-2024-08` | 70亿 | 仅限 Forge API | 生产环境，质量/速度平衡 |
| `esm3-large-2024-03` | 980亿 | 仅限 Forge API | 最高质量，研究用途 |
| `esm3-medium-multimer-2024-09` | 70亿 | 仅限 Forge API | 蛋白质复合物（实验性） |

**核心特性：**
- 跨序列、结构和功能的联合推理
- 可控步数的迭代生成
- 支持跨模态部分提示
- 复杂设计的思维链生成
- 温度控制生成多样性

## 核心 API 组件

### ESMProtein 类

表示蛋白质的核心数据结构，包含可选的序列、结构和功能信息。

**构造函数：**

```python
from esm.sdk.api import ESMProtein

protein = ESMProtein(
    sequence="MPRTKEINDAGLIVHSP",           # 氨基酸序列（可选）
    coordinates=coordinates_array,          # 三维结构（可选）
    function_annotations=[...],             # 功能标签（可选）
    secondary_structure="HHHEEEECCC",       # 二级结构标注（可选）
    sasa=sasa_array                        # 溶剂可及性（可选）
)
```

**核心方法：**

```python
# 从 PDB 文件加载
protein = ESMProtein.from_pdb("protein.pdb")

# 导出为 PDB 格式
pdb_string = protein.to_pdb()

# 保存至文件
with open("output.pdb", "w") as f:
    f.write(protein.to_pdb())
```

**掩码约定：**

使用 `_`（下划线）表示待生成位置的掩码：

```python
# 对第5-10位进行掩码生成
protein = ESMProtein(sequence="MPRT______AGLIVHSP")

# 全掩码序列（从头生成）
protein = ESMProtein(sequence="_" * 200)

# 部分结构（部分坐标为None）
protein = ESMProtein(
    sequence="MPRTKEIND",
    coordinates=partial_coords  # 部分位置可为None
)
```

### GenerationConfig 类

控制生成行为及参数。

**基础配置：**

```python
from esm.sdk.api import GenerationConfig

config = GenerationConfig(
    track="sequence",              # 生成目标："sequence"、"structure"或"function"
    num_steps=8,                  # 解掩码步数
    temperature=0.7,              # 采样温度（0.0-1.0）
    top_p=None,                   # 核心采样阈值
    condition_on_coordinates_only=False  # 结构条件约束模式
)
```

**参数详解：**

- **track**：指定生成模态
  - `"sequence"`：生成氨基酸序列
  - `"structure"`：生成三维坐标
  - `"function"`：生成功能标注

- **num_steps**：迭代解掩码步数
  - 值越高=速度越慢但质量可能更高
  - 典型范围：8-100（取决于序列长度）
  - 全序列生成建议值：约序列长度/2

- **temperature**：控制随机性
  - 0.0：完全确定性（贪婪解码）
  - 0.5-0.7：平衡探索性
  - 1.0：最大多样性
  - 值越高创新性越强，但可能降低质量

- **top_p**：核心采样参数
  - 限制采样至概率质量顶部
  - 取值范围：0.0-1.0（例如0.9=采样前90%概率质量）
  - 用于可控多样性采样

- **condition_on_coordinates_only**：结构条件约束模式
  - `True`：仅基于骨架坐标生成（忽略序列）
  - 适用于逆折叠任务

### ESM3InferenceClient 接口

本地与远程推理的统一接口。

**本地模型加载：**

```python
from esm.models.esm3 import ESM3

# 自动设备分配加载
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda")

# 或显式指定设备
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cpu")
```

**生成方法：**

```python
# 基础生成
protein_output = model.generate(protein_input, config)

# 显式指定生成目标
protein_output = model.generate(
    protein_input,
    GenerationConfig(track="sequence", num_steps=16, temperature=0.6)
)
```

**前向传播（高级）：**

```python
# 获取原始模型logits用于自定义采样
protein_tensor = model.encode(protein)
output = model.forward(protein_tensor)
logits = model.decode(output)
```

## 常用模式

### 1. 序列补全

填充蛋白质序列的掩码区域：

```python
# 定义部分序列
protein = ESMProtein(sequence="MPRTK____LIVHSP____END")

# 生成缺失位置
config = GenerationConfig(track="sequence", num_steps=12, temperature=0.5)
completed = model.generate(protein, config)

print(f"原始序列:  {protein.sequence}")
print(f"补全序列: {completed.sequence}")
```

### 2. 结构预测

从序列预测三维结构：

```python
# 输入：仅含序列
protein = ESMProtein(sequence="MPRTKEINDAGLIVHSPQWFYK")

# 生成结构
config = GenerationConfig(track="structure", num_steps=len(protein.sequence))
protein_with_structure = model.generate(protein, config)

# 保存为PDB
with open("predicted_structure.pdb", "w") as f:
    f.write(protein_with_structure.to_pdb())
```

### 3. 逆折叠

为目标结构设计序列：

```python
# 加载目标结构
target = ESMProtein.from_pdb("target.pdb")

# 移除序列，保留结构
target.sequence = None

# 生成适配该结构的序列
config = GenerationConfig(
    track="sequence",
    num_steps=50,
    temperature=0.7,
    condition_on_coordinates_only=True
)
designed = model.generate(target, config)

print(f"设计序列: {designed.sequence}")
```

### 4. 功能约束生成

生成具备特定功能的蛋白质：

```python
from esm.sdk.api import FunctionAnnotation

# 指定目标功能
protein = ESMProtein(
    sequence="_" * 150,
    function_annotations=[
        FunctionAnnotation(
            label="enzymatic_activity",
            start=30,
            end=90
        )
    ]
)

# 生成具备此功能的序列
config = GenerationConfig(track="sequence", num_steps=75, temperature=0.6)
functional_protein = model.generate(protein, config)
```

### 5. 多模态生成（思维链）

跨模态迭代生成：

```python
# 初始部分序列
protein = ESMProtein(sequence="MPRT" + "_" * 100)

# 步骤1：补全序列
protein = model.generate(
    protein,
    GenerationConfig(track="sequence", num_steps=50, temperature=0.6)
)

# 步骤2：预测完整序列的结构
protein = model.generate(
    protein,
    GenerationConfig(track="structure", num_steps=50)
)

# 步骤3：预测功能
protein = model.generate(
    protein,
    GenerationConfig(track="function", num_steps=20)
)

print(f"最终序列: {protein.sequence}")
print(f"功能标注: {protein.function_annotations}")
```

### 6. 变体生成

生成蛋白质的多种变体：

```python
import numpy as np

base_sequence = "MPRTKEINDAGLIVHSPQWFYK"
variants = []

for i in range(10):
    # 随机掩码位置
    seq_list = list(base_sequence)
    mask_indices = np.random.choice(len(seq_list), size=5, replace=False)
    for idx in mask_indices:
        seq_list[idx] = '_'

    protein = ESMProtein(sequence=''.join(seq_list))

    # 生成变体
    variant = model.generate(
        protein,
        GenerationConfig(track="sequence", num_steps=8, temperature=0.8)
    )
    variants.append(variant.sequence)

print(f"已生成 {len(variants)} 种变体")
```

## 高级主题

### 温度调度

通过动态温度提升控制效果：

```python
def generate_with_temperature_schedule(model, protein, temperatures):
    """采用降温策略进行退火生成"""
    current = protein
    steps_per_temp = 10

    for temp in temperatures:
        config = GenerationConfig(
            track="sequence",
            num_steps=steps_per_temp,
            temperature=temp
        )
        current = model.generate(current, config)

    return current

# 示例：从多样到确定
result = generate_with_temperature_schedule(
    model,
    protein,
    temperatures=[1.0, 0.8, 0.6, 0.4, 0.2]
)
```

### 约束生成

在生成过程中保留特定区域：

```python
# 固定活性位点残基
def mask_except_active_site(sequence, active_site_positions):
    """仅保留指定位置，其余掩码"""
    seq_list = ['_'] * len(sequence)
    for pos in active_site_positions:
        seq_list[pos] = sequence[pos]
    return ''.join(seq_list)

# 定义活性位点
active_site = [23, 24, 25, 45, 46, 89]
constrained_seq = mask_except_active_site(original_sequence, active_site)

protein = ESMProtein(sequence=constrained_seq)
result = model.generate(protein, GenerationConfig(track="sequence", num_steps=50))
```

### 二级结构约束

在生成中利用二级结构信息：

```python
# 定义二级结构（H=螺旋，E=折叠，C=卷曲）
protein = ESMProtein(
    sequence="_" * 80,
    secondary_structure="CCHHHHHHHEEEEECCCHHHHHHCC" + "C" * 55
)

# 生成适配该结构的序列
result = model.generate(
    protein,
    GenerationConfig(track="sequence", num_steps=40, temperature=0.6)
)
```

## 性能优化

### 内存管理

针对大型蛋白质或批量处理：

```python
import torch

# 生成间清空CUDA缓存
torch.cuda.empty_cache()

# 半精度模式节省内存
model = ESM3.from_pretrained("esm3-sm-open-v1").to("cuda").half()

# 超长序列分块处理
def chunk_generate(model, long_sequence, chunk_size=500):
    chunks = [long_sequence[i:i+chunk_size]
              for i in range(0, len(long_sequence), chunk_size)]
    results = []

    for chunk in chunks:
        protein = ESMProtein(sequence=chunk)
        result = model.generate(protein, GenerationConfig(track="sequence"))
        results.append(result.sequence)

    return ''.join(results)
```

### 批量处理建议

多蛋白质处理时：
1. 按序列长度排序实现高效批处理
2. 相似长度序列使用填充
3. 优先使用GPU处理
4. 长时任务实施检查点机制
5. 大规模处理使用Forge API（参见`forge-api.md`）

## 错误处理

```python
try:
    protein = model.generate(protein_input, config)
except ValueError as e:
    print(f"输入无效: {e}")
    # 处理无效序列或结构
except RuntimeError as e:
    print(f"生成失败: {e}")
    # 处理模型错误
except torch.cuda.OutOfMemoryError:
    print("GPU内存不足 - 尝试小型模型或CPU")
    # 回退至CPU或小型模型
```

## 模型特需考量

**esm3-sm-open-v1：**
- 适用于开发和测试
- 质量低于大型模型
- 消费级GPU快速推理
- 开放权重支持微调

**esm3-medium-2024-08：**
- 生产级质量
- 速度与精度平衡
- 需Forge API访问权限
- 推荐用于多数场景

**esm3-large-2024-03：**
- 顶尖质量
- 推理速度最慢
- 关键应用场景使用
- 创新蛋白质设计首选

## 引用

研究中使用ESM3时请引用：

```
Hayes, T. et al. (2025). Simulating 500 million years of evolution with a language model.
Science. DOI: 10.1126/science.ads0018
```
