---
name: arboreto
description: 使用可扩展算法（GRNBoost2、GENIE3）从基因表达数据推断基因调控网络（GRN）。适用于分析转录组学数据（批量RNA测序、单细胞RNA测序），以识别转录因子-靶基因关系和调控相互作用。支持大规模数据集的分布式计算。
license: BSD-3-Clause 许可证
metadata:
    skill-author: K-Dense Inc.
---

# Arboreto

## 概述

Arboreto 是一个用于从基因表达数据推断基因调控网络（GRN）的计算库，采用并行化算法，可扩展至单机或多节点集群环境。

**核心能力**：根据观测样本（细胞、样本、条件）中的表达模式，识别转录因子（TF）调控的靶基因。

## 快速入门

安装 arboreto：
```bash
uv pip install arboreto
```

基础 GRN 推断：
```python
import pandas as pd
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 加载表达数据（基因作为列）
    expression_matrix = pd.read_csv('expression_data.tsv', sep='\t')

    # 推断调控网络
    network = grnboost2(expression_data=expression_matrix)

    # 保存结果（TF, 靶基因, 重要性）
    network.to_csv('network.tsv', sep='\t', index=False, header=False)
```

**关键提示**：必须使用 `if __name__ == '__main__':` 保护语句，因为 Dask 会创建新进程。

## 核心功能

### 1. 基础 GRN 推断

标准 GRN 推断工作流包括：
- 输入数据准备（Pandas DataFrame 或 NumPy 数组）
- 使用 GRNBoost2 或 GENIE3 运行推断
- 按转录因子筛选
- 输出格式与结果解读

**参见**：`references/basic_inference.md`

**使用开箱即用脚本**：`scripts/basic_grn_inference.py` 执行标准推断任务：
```bash
python scripts/basic_grn_inference.py expression_data.tsv output_network.tsv --tf-file tfs.txt --seed 777
```

### 2. 算法选择

Arboreto 提供两种算法：

**GRNBoost2（推荐）**：
- 基于梯度提升的快速推断
- 针对大型数据集优化（10,000+ 观测样本）
- 多数分析场景的默认选择

**GENIE3**：
- 基于随机森林的推断
- 原始多元回归方法
- 适用于结果对比或验证

快速比较：
```python
from arboreto.algo import grnboost2, genie3

# 快速推荐方案
network_grnboost = grnboost2(expression_data=matrix)

# 经典算法
network_genie3 = genie3(expression_data=matrix)
```

**详细算法对比、参数说明与选择指南**：`references/algorithms.md`

### 3. 分布式计算

支持从本地多核到集群环境的扩展：

**本地模式（默认）** - 自动使用全部可用核心：
```python
network = grnboost2(expression_data=matrix)
```

**自定义本地客户端** - 资源控制：
```python
from distributed import LocalCluster, Client

local_cluster = LocalCluster(n_workers=10, memory_limit='8GB')
client = Client(local_cluster)

network = grnboost2(expression_data=matrix, client_or_address=client)

client.close()
local_cluster.close()
```

**集群计算** - 连接远程 Dask 调度器：
```python
from distributed import Client

client = Client('tcp://scheduler:8786')
network = grnboost2(expression_data=matrix, client_or_address=client)
```

**集群配置、性能优化与大规模工作流指南**：`references/distributed_computing.md`

## 安装

```bash
uv pip install arboreto
```

**依赖项**：scipy, scikit-learn, numpy, pandas, dask, distributed

## 典型应用场景

### 单细胞 RNA 测序分析
```python
import pandas as pd
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 加载单细胞表达矩阵（细胞×基因）
    sc_data = pd.read_csv('scrna_counts.tsv', sep='\t')

    # 推断细胞类型特异性调控网络
    network = grnboost2(expression_data=sc_data, seed=42)

    # 筛选高置信度调控关系
    high_confidence = network[network['importance'] > 0.5]
    high_confidence.to_csv('grn_high_confidence.tsv', sep='\t', index=False)
```

### 批量 RNA 测序与 TF 筛选
```python
from arboreto.utils import load_tf_names
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 加载数据
    expression_data = pd.read_csv('rnaseq_tpm.tsv', sep='\t')
    tf_names = load_tf_names('human_tfs.txt')

    # 在 TF 限制下进行推断
    network = grnboost2(
        expression_data=expression_data,
        tf_names=tf_names,
        seed=123
    )

    network.to_csv('tf_target_network.tsv', sep='\t', index=False)
```

### 多条件对比分析
```python
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 为不同条件推断网络
    conditions = ['control', 'treatment_24h', 'treatment_48h']

    for condition in conditions:
        data = pd.read_csv(f'{condition}_expression.tsv', sep='\t')
        network = grnboost2(expression_data=data, seed=42)
        network.to_csv(f'{condition}_network.tsv', sep='\t', index=False)
```

## 结果解读

Arboreto 返回包含调控关系的 DataFrame：

| 列名 | 描述 |
|--------|-------------|
| `TF` | 转录因子（调控因子） |
| `target` | 靶基因 |
| `importance` | 调控重要性评分（值越高表示作用越强） |

**筛选策略**：
- 每个靶基因保留前 N 个调控关系
- 重要性阈值筛选（如 > 0.5）
- 统计显著性检验（置换检验）

## 与 pySCENIC 集成

Arboreto 是单细胞调控网络分析 SCENIC 流程的核心组件：

```python
# 步骤1：使用 arboreto 进行 GRN 推断
from arboreto.algo import grnboost2
network = grnboost2(expression_data=sc_data, tf_names=tf_list)

# 步骤2：使用 pySCENIC 进行调控子识别和活性评分
# （下游分析详见 pySCENIC 文档）
```

## 结果可复现性

设置随机种子确保结果可复现：
```python
network = grnboost2(expression_data=matrix, seed=777)
```

多种子运行进行鲁棒性分析：
```python
from distributed import LocalCluster, Client

if __name__ == '__main__':
    client = Client(LocalCluster())

    seeds = [42, 123, 777]
    networks = []

    for seed in seeds:
        net = grnboost2(expression_data=matrix, client_or_address=client, seed=seed)
        networks.append(net)

    # 合并网络并筛选共识调控关系
    consensus = analyze_consensus(networks)
```

## 故障排除

**内存错误**：通过过滤低变异基因减小数据集规模，或启用分布式计算

**性能缓慢**：使用 GRNBoost2 替代 GENIE3，启用分布式客户端，筛选 TF 列表

**Dask 报错**：确保脚本中包含 `if __name__ == '__main__':` 保护语句

**结果为空**：检查数据格式（基因作为列），确认 TF 名称与基因名称匹配
