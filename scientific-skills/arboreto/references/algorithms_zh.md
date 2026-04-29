# GRN推断算法

Arboreto提供两种基因调控网络（GRN）推断算法，均基于多元回归方法。

## 算法概述

两种算法遵循相同的推断策略：
1. 为数据集中的每个目标基因训练回归模型
2. 从模型中识别最重要的特征（潜在调控因子）
3. 输出这些特征作为候选调控因子及其重要性评分

核心差异在于**计算效率**和底层回归方法。

## GRNBoost2（推荐）

**用途**：使用梯度提升实现大规模数据集的快速GRN推断。

### 适用场景
- **大型数据集**：数万次观测（如单细胞RNA测序）
- **时间敏感分析**：需要比GENIE3更快的计算结果
- **默认选择**：GRNBoost2是旗舰算法，推荐作为多数场景的首选

### 技术细节
- **方法**：带早停正则化的随机梯度提升
- **性能**：在大型数据集上显著快于GENIE3
- **输出**：与GENIE3格式相同（TF-靶基因-重要性三元组）

### 用法
```python
from arboreto.algo import grnboost2

network = grnboost2(
    expression_data=expression_matrix,
    tf_names=tf_names,
    seed=42  # 确保结果可复现
)
```

### 参数
```python
grnboost2(
    expression_data,           # 必需：pandas DataFrame或numpy数组
    gene_names=None,           # numpy数组必需提供
    tf_names='all',            # TF名称列表或'all'
    verbose=False,             # 打印进度信息
    client_or_address='local', # Dask客户端或调度器地址
    seed=None                  # 随机种子确保可复现性
)
```

## GENIE3

**用途**：基于随机森林的经典GRN推断，作为概念性基准方法。

### 适用场景
- **小型数据集**：当数据集规模允许较长计算时间时
- **对比研究**：与已发表的GENIE3结果进行对比
- **验证**：用于验证GRNBoost2结果

### 技术细节
- **方法**：随机森林或ExtraTrees回归
- **基础**：原始多元回归GRN推断策略
- **权衡**：计算成本更高但方法成熟

### 用法
```python
from arboreto.algo import genie3

network = genie3(
    expression_data=expression_matrix,
    tf_names=tf_names,
    seed=42
)
```

### 参数
```python
genie3(
    expression_data,           # 必需：pandas DataFrame或numpy数组
    gene_names=None,           # numpy数组必需提供
    tf_names='all',            # TF名称列表或'all'
    verbose=False,             # 打印进度信息
    client_or_address='local', # Dask客户端或调度器地址
    seed=None                  # 随机种子确保可复现性
)
```

## 算法对比

| 特性 | GRNBoost2 | GENIE3 |
|------|-----------|--------|
| **速度** | 快速（针对大数据优化） | 较慢 |
| **方法** | 梯度提升 | 随机森林 |
| **最佳场景** | 大规模数据（1万+观测值） | 中小型数据集 |
| **输出格式** | 相同 | 相同 |
| **推断策略** | 多元回归 | 多元回归 |
| **推荐度** | 是（默认选择） | 用于对比/验证 |

## 高级：自定义回归器参数

高级用户可传递自定义scikit-learn回归器参数：

```python
from sklearn.ensemble import GradientBoostingRegressor, RandomForestRegressor

# 自定义GRNBoost2参数
custom_grnboost2 = grnboost2(
    expression_data=expression_matrix,
    regressor_type='GBM',
    regressor_kwargs={
        'n_estimators': 100,
        'max_depth': 5,
        'learning_rate': 0.1
    }
)

# 自定义GENIE3参数
custom_genie3 = genie3(
    expression_data=expression_matrix,
    regressor_type='RF',
    regressor_kwargs={
        'n_estimators': 1000,
        'max_features': 'sqrt'
    }
)
```

## 算法选择指南

**决策流程**：

1. **首选GRNBoost2** - 速度更快且更擅长处理大型数据集
2. **选用GENIE3当**：
   - 与现有GENIE3文献对比
   - 数据集为中小规模
   - 验证GRNBoost2结果

两种算法生成的可比调控网络具有相同输出格式，在多数分析中可互换使用。
