# COBRApy API 快速参考

本文档提供常用 COBRApy 函数、签名和使用模式的快速参考。

## 模型输入输出

### 加载模型

```python
from cobra.io import load_model, read_sbml_model, load_json_model, load_yaml_model, load_matlab_model

# 内置测试模型
model = load_model("textbook")   # 大肠杆菌核心代谢
model = load_model("ecoli")      # 完整大肠杆菌 iJO1366
model = load_model("salmonella") # 沙门氏菌 LT2

# 从文件加载
model = read_sbml_model(filename, f_replace={}, **kwargs)
model = load_json_model(filename)
model = load_yaml_model(filename)
model = load_matlab_model(filename, variable_name=None)
```

### 保存模型

```python
from cobra.io import write_sbml_model, save_json_model, save_yaml_model, save_matlab_model

write_sbml_model(model, filename, f_replace={}, **kwargs)
save_json_model(model, filename, pretty=False, **kwargs)
save_yaml_model(model, filename, **kwargs)
save_matlab_model(model, filename, **kwargs)
```

## 模型结构

### 核心类

```python
from cobra import Model, Reaction, Metabolite, Gene

# 创建模型
model = Model(id_or_model=None, name=None)

# 创建代谢物
metabolite = Metabolite(
    id=None,
    formula=None,
    name="",
    charge=None,
    compartment=None
)

# 创建反应
reaction = Reaction(
    id=None,
    name="",
    subsystem="",
    lower_bound=0.0,
    upper_bound=None
)

# 创建基因
gene = Gene(id=None, name="", functional=True)
```

### 模型属性

```python
# 组件访问（DictList 对象）
model.reactions       # Reaction 对象的 DictList
model.metabolites     # Metabolite 对象的 DictList
model.genes          # Gene 对象的 DictList

# 特殊反应列表
model.exchanges      # 交换反应（外部转运）
model.demands        # 需求反应（代谢物汇）
model.sinks          # 汇反应
model.boundary       # 所有边界反应

# 模型属性
model.objective      # 当前目标函数（可读写）
model.objective_direction  # "max" 或 "min"
model.medium         # 生长培养基（exchange: bound 字典）
model.solver         # 优化求解器
```

### DictList 方法

```python
# 按索引访问
item = model.reactions[0]

# 按 ID 访问
item = model.reactions.get_by_id("PFK")

# 字符串查询（子串匹配）
items = model.reactions.query("atp")      # 不区分大小写搜索
items = model.reactions.query(lambda x: x.subsystem == "Glycolysis")

# 列表推导式
items = [r for r in model.reactions if r.lower_bound < 0]

# 成员检查
"PFK" in model.reactions
```

## 优化

### 基础优化

```python
# 完整优化（返回 Solution 对象）
solution = model.optimize()

# Solution 属性
solution.objective_value   # 目标函数值
solution.status           # 优化状态（"optimal"、"infeasible" 等）
solution.fluxes          # 反应通量的 Pandas Series
solution.shadow_prices   # 代谢物影子价格的 Pandas Series
solution.reduced_costs   # 降低成本的 Pandas Series

# 快速优化（仅返回浮点数）
objective_value = model.slim_optimize()

# 更改目标函数
model.objective = "ATPM"
model.objective = model.reactions.ATPM
model.objective = {model.reactions.ATPM: 1.0}

# 更改优化方向
model.objective_direction = "max"  # 或 "min"
```

### 求解器配置

```python
# 检查可用求解器
from cobra.util.solver import solvers
print(solvers)

# 更改求解器
model.solver = "glpk"  # 或 "cplex"、"gurobi" 等

# 求解器特定配置
model.solver.configuration.timeout = 60  # 秒
model.solver.configuration.verbosity = 1
model.solver.configuration.tolerances.feasibility = 1e-9
```

## 通量分析

### 通量平衡分析 (FBA)

```python
from cobra.flux_analysis import pfba, geometric_fba

# 简约 FBA
solution = pfba(model, fraction_of_optimum=1.0, **kwargs)

# 几何 FBA
solution = geometric_fba(model, epsilon=1e-06, max_tries=200)
```

### 通量变异性分析 (FVA)

```python
from cobra.flux_analysis import flux_variability_analysis

fva_result = flux_variability_analysis(
    model,
    reaction_list=None,        # 反应 ID 列表（None 表示全部）
    loopless=False,            # 消除热力学不可行循环
    fraction_of_optimum=1.0,   # 最优比例 (0.0-1.0)
    pfba_factor=None,          # 可选 pFBA 约束
    processes=1                # 并行进程数
)

# 返回包含列：minimum, maximum 的 DataFrame
```

### 基因和反应删除

```python
from cobra.flux_analysis import (
    single_gene_deletion,
    single_reaction_deletion,
    double_gene_deletion,
    double_reaction_deletion
)

# 单删除
results = single_gene_deletion(
    model,
    gene_list=None,     # None 表示所有基因
    processes=1,
    **kwargs
)

results = single_reaction_deletion(
    model,
    reaction_list=None,  # None 表示所有反应
    processes=1,
    **kwargs
)

# 双删除
results = double_gene_deletion(
    model,
    gene_list1=None,
    gene_list2=None,
    processes=1,
    **kwargs
)

results = double_reaction_deletion(
    model,
    reaction_list1=None,
    reaction_list2=None,
    processes=1,
    **kwargs
)

# 返回包含列：ids, growth, status 的 DataFrame
# 双删除索引为基因/反应对的 MultiIndex
```

### 通量采样

```python
from cobra.sampling import sample, OptGPSampler, ACHRSampler

# 简单接口
samples = sample(
    model,
    n,                  # 样本数量
    method="optgp",     # 或 "achr"
    thinning=100,       # 稀释因子（每 n 次迭代采样）
    processes=1,        # 并行进程（仅 OptGP）
    seed=None          # 随机种子
)

# 高级接口（采样器对象）
sampler = OptGPSampler(model, processes=4, thinning=100)
sampler = ACHRSampler(model, thinning=100)

# 生成样本
samples = sampler.sample(n)

# 验证样本
validation = sampler.validate(sampler.samples)
# 返回数组：'v'（有效）、'l'（下界违规）、
# 'u'（上界违规）、'e'（等式违规）

# 批量采样
sampler.batch(n_samples, n_batches)
```

### 生产包络线

```python
from cobra.flux_analysis import production_envelope

envelope = production_envelope(
    model,
    reactions,              # 1-2 个反应 ID 的列表
    objective=None,         # 目标反应 ID（None 使用模型目标）
    carbon_sources=None,    # 用于产量计算的碳源
    points=20,              # 计算点数
    threshold=0.01          # 最小目标值阈值
)

# 返回包含列的 DataFrame：
# - 第一个反应通量
# - 第二个反应通量（如提供）
# - objective_minimum, objective_maximum
# - carbon_yield_minimum, carbon_yield_maximum（如指定碳源）
# - mass_yield_minimum, mass_yield_maximum
```

### 间隙填充

```python
from cobra.flux_analysis import gapfill

# 基础间隙填充
solution = gapfill(
    model,
    universal=None,         # 包含候选反应的通用模型
    lower_bound=0.05,       # 最小目标通量
    penalties=None,         # 反应:惩罚 字典
    demand_reactions=True,  # 需要时添加需求反应
    exchange_reactions=False,
    iterations=1
)

# 返回要添加的 Reaction 对象列表

# 多解决方案
solutions = []
for i in range(5):
    sol = gapfill(model, universal, iterations=1)
    solutions.append(sol)
    # 通过增加惩罚避免找到相同解
```

### 其他分析方法

```python
from cobra.flux_analysis import (
    find_blocked_reactions,
    find_essential_genes,
    find_essential_reactions
)

# 阻塞反应（无法承载通量）
blocked = find_blocked_reactions(
    model,
    reaction_list=None,
    zero_cutoff=1e-9,
    open_exchanges=False
)

# 必需基因/反应
essential_genes = find_essential_genes(model, threshold=0.01)
essential_reactions = find_essential_reactions(model, threshold=0.01)
```

## 培养基与边界条件

### 培养基管理

```python
# 获取当前培养基（返回字典）
medium = model.medium

# 设置培养基（必须重新分配整个字典）
medium = model.medium
medium["EX_glc__D_e"] = 10.0
medium["EX_o2_e"] = 20.0
model.medium = medium

# 替代方案：单独修改
with model:
    model.reactions.EX_glc__D_e.lower_bound = -10.0
```

### 最小培养基

```python
from cobra.medium import minimal_medium

min_medium = minimal_medium(
    model,
    min_objective_value=0.1,  # 最小生长速率
    minimize_components=False, # 为 True 时使用 MILP（较慢）
    open_exchanges=False,      # 优化前开放所有交换
    exports=False,             # 允许代谢物输出
    penalties=None             # exchange:惩罚 字典
)

# 返回带通量的交换反应 Series
```

### 边界反应

```python
# 添加边界反应
model.add_boundary(
    metabolite,
    type="exchange",    # 或 "demand"、"sink"
    reaction_id=None,   # 为 None 时自动生成
    lb=None,
    ub=None,
    sbo_term=None
)

# 访问边界反应
exchanges = model.exchanges     # 系统边界
demands = model.demands         # 胞内移除
sinks = model.sinks            # 胞内交换
boundaries = model.boundary    # 所有边界反应
```

## 模型操作

### 添加组件

```python
# 添加反应
model.add_reactions([reaction1, reaction2, ...])
model.add_reaction(reaction)

# 添加代谢物
reaction.add_metabolites({
    metabolite1: -1.0,  # 消耗（负化学计量）
    metabolite2: 1.0    # 产生（正化学计量）
})

# 向模型添加代谢物
model.add_metabolites([metabolite1, metabolite2, ...])

# 添加基因（通常通过 gene_reaction_rule 自动添加）
model.genes += [gene1, gene2, ...]
```

### 移除组件

```python
# 移除反应
model.remove_reactions([reaction1, reaction2, ...])
model.remove_reactions(["PFK", "FBA"])

# 移除代谢物（同时从反应中移除）
model.remove_metabolites([metabolite1, metabolite2, ...])

# 移除基因（通常通过 gene_reaction_rule）
model.genes.remove(gene)
```

### 修改反应

```python
# 设置边界
reaction.bounds = (lower, upper)
reaction.lower_bound = 0.0
reaction.upper_bound = 1000.0

# 修改化学计量
reaction.add_metabolites({metabolite: 1.0})
reaction.subtract_metabolites({metabolite: 1.0})

# 更改基因-反应规则
reaction.gene_reaction_rule = "(gene1 and gene2) or gene3"

# 敲除
reaction.knock_out()
gene.knock_out()
```

### 模型复制

```python
# 深拷贝（独立模型）
model_copy = model.copy()

# 复制特定反应
new_model = Model("subset")
reactions_to_copy = [model.reactions.PFK, model.reactions.FBA]
new_model.add_reactions(reactions_to_copy)
```

## 上下文管理

使用上下文管理器进行临时修改：

```python
# 在 with 块后自动恢复更改
with model:
    model.objective = "ATPM"
    model.reactions.EX_glc__D_e.lower_bound = -5.0
    model.genes.b0008.knock_out()
    solution = model.optimize()

# 此处模型状态已恢复

# 嵌套多上下文
with model:
    model.objective = "ATPM"
    with model:
        model.genes.b0008.knock_out()
        # 两项修改均生效
    # 仅目标函数更改生效

# 反应上下文管理
with model:
    model.reactions.PFK.knock_out()
    # 等效于：reaction.lower_bound = reaction.upper_bound = 0
```

## 反应与代谢物属性

### 反应属性

```python
reaction.id                      # 唯一标识符
reaction.name                    # 可读名称
reaction.subsystem               # 通路/子系统
reaction.bounds                  # (lower_bound, upper_bound)
reaction.lower_bound
reaction.upper_bound
reaction.reversibility          # 布尔值 (lower_bound < 0)
reaction.gene_reaction_rule     # GPR 字符串
reaction.genes                  # 关联 Gene 对象集合
reaction.metabolites            # {代谢物: 化学计量} 字典

# 方法
reaction.reaction               # 化学计量方程字符串
reaction.build_reaction_string() # 同上
reaction.check_mass_balance()   # 返回不平衡或空字典
reaction.get_coefficient(metabolite_id)
reaction.add_metabolites({metabolite: coeff})
reaction.subtract_metabolites({metabolite: coeff})
reaction.knock_out()
```

### 代谢物属性

```python
metabolite.id                   # 唯一标识符
metabolite.name                 # 可读名称
metabolite.formula              # 化学式
metabolite.charge               # 电荷
metabolite.compartment          # 区室 ID
metabolite.reactions            # 关联反应 FrozenSet

# 方法
metabolite.summary()            # 打印生产/消耗摘要
metabolite.copy()
```

### 基因属性

```python
gene.id                         # 唯一标识符
gene.name                       # 可读名称
gene.functional                 # 布尔活性状态
gene.reactions                  # 关联反应 FrozenSet

# 方法
gene.knock_out()
```

## 模型验证

### 一致性检查

```python
from cobra.manipulation import check_mass_balance, check_metabolite_compartment_formula

# 检查所有反应的质量平衡
unbalanced = {}
for reaction in model.reactions:

```markdown
"growth": solution.objective_value if solution.status == "optimal" else 0,
            "status": solution.status
        })

df = pd.DataFrame(knockout_results)
```

### 参数扫描模式

```python
parameter_values = np.linspace(0, 20, 21)
results = []

for value in parameter_values:
    with model:
        model.reactions.EX_glc__D_e.lower_bound = -value

        solution = model.optimize()

        results.append({
            "glucose_uptake": value,
            "growth": solution.objective_value,
            "acetate_secretion": solution.fluxes["EX_ac_e"]
        })

df = pd.DataFrame(results)
```

本快速参考涵盖了最常用的 COBRApy 函数和模式。完整的 API 文档请参见 https://cobrapy.readthedocs.io/
```
