# TDC 分子生成评估函数

评估函数用于在特定维度上衡量生成分子的质量。TDC 为药物从头设计中的分子优化任务提供了 17 种以上的评估函数。

## 概述

评估函数测量分子属性，主要服务于两个目标：

1. **目标导向生成**：优化分子以最大化/最小化特定属性
2. **分布学习**：评估生成分子是否符合期望的属性分布

## 使用评估函数

### 基础用法

```python
from tdc import Oracle

# 初始化评估函数
oracle = Oracle(name='GSK3B')

# 评估单个分子（SMILES 字符串）
score = oracle('CC(C)Cc1ccc(cc1)C(C)C(O)=O')

# 评估多个分子
scores = oracle(['SMILES1', 'SMILES2', 'SMILES3'])
```

### 评估函数分类

TDC 评估函数根据评估的分子属性分为多个类别。

## 生物化学评估函数

预测与生物靶点的结合亲和力或活性。

### 靶点特异性评估函数

**DRD2 - 多巴胺受体 D2**
```python
oracle = Oracle(name='DRD2')
score = oracle(smiles)
```
- 测量与 DRD2 受体的结合亲和力
- 对神经和精神疾病药物开发至关重要
- 分数越高表示结合力越强

**GSK3B - 糖原合成酶激酶-3β**
```python
oracle = Oracle(name='GSK3B')
score = oracle(smiles)
```
- 预测 GSK3β 抑制效果
- 与阿尔茨海默症、糖尿病和癌症研究相关
- 分数越高表示抑制效果越好

**JNK3 - c-Jun 氨基末端激酶 3**
```python
oracle = Oracle(name='JNK3')
score = oracle(smiles)
```
- 测量 JNK3 激酶抑制效果
- 神经退行性疾病治疗靶点
- 分数越高表示抑制力越强

**5HT2A - 血清素 2A 受体**
```python
oracle = Oracle(name='5HT2A')
score = oracle(smiles)
```
- 预测血清素受体结合力
- 对精神疾病药物至关重要
- 分数越高表示结合力越强

**ACE - 血管紧张素转化酶**
```python
oracle = Oracle(name='ACE')
score = oracle(smiles)
```
- 测量 ACE 抑制效果
- 高血压治疗靶点
- 分数越高表示抑制效果越好

**MAPK - 丝裂原活化蛋白激酶**
```python
oracle = Oracle(name='MAPK')
score = oracle(smiles)
```
- 预测 MAPK 抑制效果
- 癌症和炎症疾病治疗靶点

**CDK - 细胞周期蛋白依赖性激酶**
```python
oracle = Oracle(name='CDK')
score = oracle(smiles)
```
- 测量 CDK 抑制效果
- 对癌症药物开发至关重要

**P38 - p38 MAP 激酶**
```python
oracle = Oracle(name='P38')
score = oracle(smiles)
```
- 预测 p38 MAPK 抑制效果
- 炎症疾病治疗靶点

**PARP1 - 聚(ADP-核糖)聚合酶 1**
```python
oracle = Oracle(name='PARP1')
score = oracle(smiles)
```
- 测量 PARP1 抑制效果
- 癌症治疗靶点（DNA 修复机制）

**PIK3CA - 磷脂酰肌醇-4,5-二磷酸 3-激酶**
```python
oracle = Oracle(name='PIK3CA')
score = oracle(smiles)
```
- 预测 PIK3CA 抑制效果
- 肿瘤学重要靶点

## 物理化学评估函数

评估类药特性和 ADME 特征。

### 类药性评估函数

**QED - 类药性定量评估**
```python
oracle = Oracle(name='QED')
score = oracle(smiles)
```
- 综合多种物理化学属性
- 分数范围 0（非类药）至 1（类药）
- 基于 Bickerton 等人标准

**Lipinski - 五规则**
```python
oracle = Oracle(name='Lipinski')
score = oracle(smiles)
```
- 违反 Lipinski 规则的次数
- 规则：分子量 ≤ 500，logP ≤ 5，氢键供体 ≤ 5，氢键受体 ≤ 10
- 分数为 0 表示完全合规

### 分子属性

**SA - 合成可及性**
```python
oracle = Oracle(name='SA')
score = oracle(smiles)
```
- 评估合成难易度
- 分数范围 1（易合成）至 10（难合成）
- 分数越低表示越易合成

**LogP - 辛醇/水分配系数**
```python
oracle = Oracle(name='LogP')
score = oracle(smiles)
```
- 测量亲脂性
- 对细胞膜渗透性至关重要
- 典型类药范围：0-5

**MW - 分子量**
```python
oracle = Oracle(name='MW')
score = oracle(smiles)
```
- 返回分子量（道尔顿）
- 典型类药范围：150-500 Da

## 复合评估函数

组合多个属性实现多目标优化。

**异构体元数据**
```python
oracle = Oracle(name='Isomer_Meta')
score = oracle(smiles)
```
- 评估特定异构体属性
- 用于立体化学优化

**中值分子**
```python
oracle = Oracle(name='Median1', 'Median2')
score = oracle(smiles)
```
- 测试生成具有中值属性分子的能力
- 适用于分布学习基准测试

**再发现**
```python
oracle = Oracle(name='Rediscovery')
score = oracle(smiles)
```
- 测量与已知参考分子的相似度
- 测试重现现有药物的能力

**相似度**
```python
oracle = Oracle(name='Similarity')
score = oracle(smiles)
```
- 计算与目标分子的结构相似度
- 基于分子指纹（通常为 Tanimoto 相似度）

**独特性**
```python
oracle = Oracle(name='Uniqueness')
scores = oracle(smiles_list)
```
- 衡量生成分子集的多样性
- 返回独特分子的比例

**新颖性**
```python
oracle = Oracle(name='Novelty')
scores = oracle(smiles_list, training_set)
```
- 衡量生成分子与训练集的差异度
- 分数越高表示结构越新颖

## 专用评估函数

**ASKCOS - 逆合成评分**
```python
oracle = Oracle(name='ASKCOS')
score = oracle(smiles)
```
- 通过逆合成评估合成可行性
- 需要 ASKCOS 后端（IBM RXN）
- 基于逆合成路径可用性评分

**对接分数**
```python
oracle = Oracle(name='Docking')
score = oracle(smiles)
```
- 针对靶蛋白的分子对接分数
- 需要蛋白质结构和对接软件
- 分数越低通常表示结合越好

**Vina - AutoDock Vina 分数**
```python
oracle = Oracle(name='Vina')
score = oracle(smiles)
```
- 使用 AutoDock Vina 进行蛋白-配体对接
- 预测结合亲和力（单位 kcal/mol）
- 负值越大表示结合越强

## 多目标优化

组合多个评估函数实现多属性优化：

```python
from tdc import Oracle

# 初始化多个评估函数
qed_oracle = Oracle(name='QED')
sa_oracle = Oracle(name='SA')
drd2_oracle = Oracle(name='DRD2')

# 定义自定义评分函数
def multi_objective_score(smiles):
    qed = qed_oracle(smiles)
    sa = 1 / (1 + sa_oracle(smiles))  # SA 取倒数（值越低越好）
    drd2 = drd2_oracle(smiles)

    # 加权组合
    return 0.3 * qed + 0.3 * sa + 0.4 * drd2

# 评估分子
score = multi_objective_score('CC(C)Cc1ccc(cc1)C(C)C(O)=O')
```

## 评估函数性能考量

### 速度
- **快速**：QED, SA, LogP, MW, Lipinski（基于规则计算）
- **中等**：靶点特异性 ML 模型（DRD2, GSK3B 等）
- **慢速**：基于对接的评估函数（Vina, ASKCOS）

### 可靠性
- 评估函数是在特定数据集上训练的 ML 模型
- 可能无法泛化到所有化学空间
- 建议使用多个评估函数验证结果

### 批量处理
```python
# 高效批量评估
oracle = Oracle(name='GSK3B')
smiles_list = ['SMILES1', 'SMILES2', ..., 'SMILES1000']
scores = oracle(smiles_list)  # 比单次调用更快
```

## 常见工作流

### 目标导向生成
```python
from tdc import Oracle
from tdc.generation import MolGen

# 加载训练数据
data = MolGen(name='ChEMBL_V29')
train_smiles = data.get_data()['Drug'].tolist()

# 初始化评估函数
oracle = Oracle(name='GSK3B')

# 生成分子（用户实现生成模型）
# generated_smiles = generator.generate(n=1000)

# 评估生成分子
scores = oracle(generated_smiles)
best_molecules = [(s, score) for s, score in zip(generated_smiles, scores)]
best_molecules.sort(key=lambda x: x[1], reverse=True)

print(f"Top 10 molecules:")
for smiles, score in best_molecules[:10]:
    print(f"{smiles}: {score:.3f}")
```

### 分布学习
```python
from tdc import Oracle
import numpy as np

# 初始化评估函数
oracle = Oracle(name='QED')

# 评估训练集
train_scores = oracle(train_smiles)
train_mean = np.mean(train_scores)
train_std = np.std(train_scores)

# 评估生成集
gen_scores = oracle(generated_smiles)
gen_mean = np.mean(gen_scores)
gen_std = np.std(gen_scores)

# 比较分布
print(f"训练集: μ={train_mean:.3f}, σ={train_std:.3f}")
print(f"生成集: μ={gen_mean:.3f}, σ={gen_std:.3f}")
```

## 与 TDC 基准集成

```python
from tdc.generation import MolGen

# 用于 GuacaMol 基准测试
data = MolGen(name='GuacaMol')

# 评估函数自动集成
# 每个 GuacaMol 任务关联对应评估函数
benchmark_results = data.evaluate_guacamol(
    generated_molecules=your_molecules,
    oracle_name='GSK3B'
)
```

## 注意事项

- 评估函数分数为预测值，非实验测量结果
- 顶级候选分子需实验验证
- 不同评估函数可能有不同的分数范围和解释
- 部分评估函数需额外依赖或 API 访问权限
- 具体细节请查阅评估函数文档：https://tdcommons.ai/functions/oracles/

## 添加自定义评估函数

创建自定义评估函数：

```python
class CustomOracle:
    def __init__(self):
        # 初始化模型/方法
        pass

    def __call__(self, smiles):
        # 实现评分逻辑
        # 返回分数或分数列表
        pass

# 用法与内置评估函数相同
custom_oracle = CustomOracle()
score = custom_oracle('CC(C)Cc1ccc(cc1)C(C)C(O)=O')
```

## 参考文献

- TDC 评估函数文档：https://tdcommons.ai/functions/oracles/
- GuacaMol 论文："GuacaMol: Benchmarking Models for de Novo Molecular Design"
- MOSES 论文："Molecular Sets (MOSES): A Benchmarking Platform for Molecular Generation Models"
