---
name: cobrapy
description: 基于约束的代谢建模（COBRA）。支持通量平衡分析（FBA）、通量变异性分析（FVA）、基因敲除、通量采样、SBML模型，用于系统生物学和代谢工程分析。
license: GPL-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# COBRApy - 基于约束的重建与分析

## 概述

COBRApy 是一个用于代谢模型约束式重建与分析（COBRA）的 Python 库，是系统生物学研究的关键工具。支持基因组尺度代谢模型，可执行细胞代谢计算模拟、代谢工程分析及表型行为预测。

## 核心功能

COBRApy 提供涵盖多个关键领域的综合工具：

### 1. 模型管理

从仓库或文件加载现有模型：
```python
from cobra.io import load_model

# 加载内置测试模型
model = load_model("textbook")  # 大肠杆菌核心模型
model = load_model("ecoli")     # 完整大肠杆菌模型
model = load_model("salmonella")

# 从文件加载
from cobra.io import read_sbml_model, load_json_model, load_yaml_model
model = read_sbml_model("path/to/model.xml")
model = load_json_model("path/to/model.json")
model = load_yaml_model("path/to/model.yml")
```

以多种格式保存模型：
```python
from cobra.io import write_sbml_model, save_json_model, save_yaml_model
write_sbml_model(model, "output.xml")  # 首选格式
save_json_model(model, "output.json")  # 兼容 Escher
save_yaml_model(model, "output.yml")   # 人类可读格式
```

### 2. 模型结构与组件

访问和检查模型组件：
```python
# 访问组件
model.reactions      # 所有反应的 DictList
model.metabolites    # 所有代谢物的 DictList
model.genes          # 所有基因的 DictList

# 通过 ID 或索引获取特定项
reaction = model.reactions.get_by_id("PFK")
metabolite = model.metabolites[0]

# 检查属性
print(reaction.reaction)        # 化学计量方程
print(reaction.bounds)          # 通量约束范围
print(reaction.gene_reaction_rule)  # 基因反应规则（GPR逻辑）
print(metabolite.formula)       # 化学式
print(metabolite.compartment)   # 细胞定位
```

### 3. 通量平衡分析（FBA）

执行标准 FBA 模拟：
```python
# 基础优化
solution = model.optimize()
print(f"目标值: {solution.objective_value}")
print(f"状态: {solution.status}")

# 访问通量值
print(solution.fluxes["PFK"])
print(solution.fluxes.head())

# 快速优化（仅获取目标值）
objective_value = model.slim_optimize()

# 更改目标函数
model.objective = "ATPM"
solution = model.optimize()
```

简约 FBA（最小化总通量）：
```python
from cobra.flux_analysis import pfba
solution = pfba(model)
```

几何 FBA（寻找中心解）：
```python
from cobra.flux_analysis import geometric_fba
solution = geometric_fba(model)
```

### 4. 通量变异性分析（FVA）

确定所有反应的通量范围：
```python
from cobra.flux_analysis import flux_variability_analysis

# 标准 FVA
fva_result = flux_variability_analysis(model)

# 90%最优性下的 FVA
fva_result = flux_variability_analysis(model, fraction_of_optimum=0.9)

# 无循环 FVA（消除热力学不可行循环）
fva_result = flux_variability_analysis(model, loopless=True)

# 特定反应的 FVA
fva_result = flux_variability_analysis(
    model,
    reaction_list=["PFK", "FBA", "PGI"]
)
```

### 5. 基因与反应缺失研究

执行敲除分析：
```python
from cobra.flux_analysis import (
    single_gene_deletion,
    single_reaction_deletion,
    double_gene_deletion,
    double_reaction_deletion
)

# 单基因敲除
gene_results = single_gene_deletion(model)
reaction_results = single_reaction_deletion(model)

# 双基因敲除（使用多进程）
double_gene_results = double_gene_deletion(
    model,
    processes=4  # CPU核心数
)

# 使用上下文管理器手动敲除
with model:
    model.genes.get_by_id("b0008").knock_out()
    solution = model.optimize()
    print(f"敲除后生长率: {solution.objective_value}")
# 退出上下文后模型自动恢复
```

### 6. 生长培养基与最小培养基

管理生长培养基：
```python
# 查看当前培养基
print(model.medium)

# 修改培养基（需重新赋值整个字典）
medium = model.medium
medium["EX_glc__D_e"] = 10.0  # 设置葡萄糖摄取
medium["EX_o2_e"] = 0.0       # 厌氧条件
model.medium = medium

# 计算最小培养基
from cobra.medium import minimal_medium

# 最小化总输入通量
min_medium = minimal_medium(model, minimize_components=False)

# 最小化组分数量（使用 MILP，较慢）
min_medium = minimal_medium(
    model,
    minimize_components=True,
    open_exchanges=True
)
```

### 7. 通量采样

采样可行通量空间：
```python
from cobra.sampling import sample

# 使用 OptGP 采样（默认，支持并行处理）
samples = sample(model, n=1000, method="optgp", processes=4)

# 使用 ACHR 采样
samples = sample(model, n=1000, method="achr")

# 验证采样结果
from cobra.sampling import OptGPSampler
sampler = OptGPSampler(model, processes=4)
sampler.sample(1000)
validation = sampler.validate(sampler.samples)
print(validation.value_counts())  # 应全为 'v'（有效）
```

### 8. 生产包络线

计算表型相平面：
```python
from cobra.flux_analysis import production_envelope

# 标准生产包络线
envelope = production_envelope(
    model,
    reactions=["EX_glc__D_e", "EX_o2_e"],
    objective="EX_ac_e"  # 乙酸生产
)

# 含碳源得率
envelope = production_envelope(
    model,
    reactions=["EX_glc__D_e", "EX_o2_e"],
    carbon_sources="EX_glc__D_e"
)

# 可视化（使用 matplotlib 或 pandas 绘图）
import matplotlib.pyplot as plt
envelope.plot(x="EX_glc__D_e", y="EX_o2_e", kind="scatter")
plt.show()
```

### 9. 间隙填充

添加反应使模型可行：
```python
from cobra.flux_analysis import gapfill

# 准备含候选反应的通用模型
universal = load_model("universal")

# 执行间隙填充
with model:
    # 移除反应以创建间隙（演示用）
    model.remove_reactions([model.reactions.PGI])

    # 查找需添加的反应
    solution = gapfill(model, universal)
    print(f"需添加的反应: {solution}")
```

### 10. 模型构建

从零构建模型：
```python
from cobra import Model, Reaction, Metabolite

# 创建模型
model = Model("my_model")

# 创建代谢物
atp_c = Metabolite("atp_c", formula="C10H12N5O13P3",
                   name="ATP", compartment="c")
adp_c = Metabolite("adp_c", formula="C10H12N5O10P2",
                   name="ADP", compartment="c")
pi_c = Metabolite("pi_c", formula="HO4P",
                  name="磷酸盐", compartment="c")

# 创建反应
reaction = Reaction("ATPASE")
reaction.name = "ATP水解"
reaction.subsystem = "能量代谢"
reaction.lower_bound = 0.0
reaction.upper_bound = 1000.0

# 添加代谢物及化学计量系数
reaction.add_metabolites({
    atp_c: -1.0,
    adp_c: 1.0,
    pi_c: 1.0
})

# 添加基因反应规则
reaction.gene_reaction_rule = "(gene1 and gene2) or gene3"

# 添加到模型
model.add_reactions([reaction])

# 添加边界反应
model.add_boundary(atp_c, type="exchange")
model.add_boundary(adp_c, type="demand")

# 设置目标函数
model.objective = "ATPASE"
```

## 常用工作流

### 工作流 1：加载模型并预测生长

```python
from cobra.io import load_model

# 加载模型
model = load_model("ecoli")

# 执行 FBA
solution = model.optimize()
print(f"生长速率: {solution.objective_value:.3f} /h")

# 显示活跃通路
print(solution.fluxes[solution.fluxes.abs() > 1e-6])
```

### 工作流 2：基因敲除筛选

```python
from cobra.io import load_model
from cobra.flux_analysis import single_gene_deletion

# 加载模型
model = load_model("ecoli")

# 执行单基因敲除
results = single_gene_deletion(model)

# 查找必需基因（生长率 < 阈值）
essential_genes = results[results["growth"] < 0.01]
print(f"发现 {len(essential_genes)} 个必需基因")

# 查找影响最小的基因
neutral_genes = results[results["growth"] > 0.9 * solution.objective_value]
```

### 工作流 3：培养基优化

```python
from cobra.io import load_model
from cobra.medium import minimal_medium

# 加载模型
model = load_model("ecoli")

# 计算最大生长率50%的最小培养基
target_growth = model.slim_optimize() * 0.5
min_medium = minimal_medium(
    model,
    target_growth,
    minimize_components=True
)

print(f"最小培养基组分数量: {len(min_medium)}")
print(min_medium)
```

### 工作流 4：通量不确定性分析

```python
from cobra.io import load_model
from cobra.flux_analysis import flux_variability_analysis
from cobra.sampling import sample

# 加载模型
model = load_model("ecoli")

# 首先检查最优状态下的通量范围
fva = flux_variability_analysis(model, fraction_of_optimum=1.0)

# 对通量范围大的反应进行采样以理解分布
samples = sample(model, n=1000)

# 分析特定反应
reaction_id = "PFK"
import matplotlib.pyplot as plt
samples[reaction_id].hist(bins=50)
plt.xlabel(f"{reaction_id} 通量分布")
plt.ylabel("频率")
plt.show()
```

### 工作流 5：使用上下文管理器进行临时修改

通过上下文管理器实现临时修改：
```python
# 模型在上下文外保持不变
with model:
    # 临时更改目标函数
    model.objective = "ATPM"

    # 临时修改边界
    model.reactions.EX_glc__D_e.lower_bound = -5.0

    # 临时敲除基因
    model.genes.b0008.knock_out()

    # 在修改状态下优化
    solution = model.optimize()
    print(f"修改后生长率: {solution.objective_value}")

# 所有修改自动还原
solution = model.optimize()
print(f"原始生长率: {solution.objective_value}")
```

## 关键概念

### DictList 对象
模型使用 `DictList` 对象管理反应、代谢物和基因，兼具列表和字典特性：
```python
# 按索引访问
first_reaction = model.reactions[0]

# 按 ID 访问
pfk = model.reactions.get_by_id("PFK")

# 查询方法
atp_reactions = model.reactions.query("atp")
```

### 通量约束
反应边界定义可行通量范围：
- **不可逆反应**：`lower_bound = 0, upper_bound > 0`
- **可逆反应**：`lower_bound < 0, upper_bound > 0`
- 使用 `.bounds` 同时设置边界以避免不一致

### 基因反应规则（GPR）
连接基因与反应的布尔逻辑：
```python
# AND 逻辑（两者必需）
reaction.gene_reaction_rule = "gene1 and gene2"

# OR 逻辑（任一满足）
reaction.gene_reaction_rule = "gene1 or gene2"

# 复合逻辑
reaction.gene_reaction_rule = "(gene1 and gene2) or (gene3 and gene4)"
```

### 交换反应
代表代谢物输入/输出的特殊反应：
- 命名约定前缀 `EX_`
- 正通量 = 分泌，负通量 = 摄取
- 通过 `model.medium` 字典管理

## 最佳实践

1. **使用上下文管理器**进行临时修改，避免状态管理问题
2. **优化前验证模型**：使用 `model.slim_optimize()` 确保可行性
3. **检查优化状态**：`optimal` 表示成功求解
4. **涉及热力学可行性时使用无循环 FVA**
5. **在 FVA 中合理设置 fraction_of_optimum** 以探索次优空间
6. **并行化计算密集型操作**（采样、双敲除）
7. **模型交换与长期存储首选 SBML 格式**
8. **仅需目标值时使用 slim_optimize()** 提升性能
9. **验证通量采样结果** 确保数值稳定性

## 故障排除

**不可行解**：检查培养基约束、反应边界和模型一致性  
**优化缓慢**：尝试不同求解器（GLPK/CPLEX/Gurobi）：`model.solver`  
**无界解**：确认交换反应有合理上界  
**导入错误**：确保文件格式正确且 SBML 标识符有效  

## 参考文献

详细工作流和 API 模式参考：  
- `references/workflows.md` - 分步工作流示例  
- `references/api_quick_reference.md` - 常用函数签名与模式  

官方文档：https://cobrapy.readthedocs.io/en/latest/
