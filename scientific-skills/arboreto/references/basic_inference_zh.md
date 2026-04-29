# 使用 Arboreto 进行基础 GRN 推断

## 输入数据要求

Arboreto 要求基因表达数据为以下两种格式之一：

### Pandas 数据框（推荐）
- **行**：观测值（细胞、样本、条件）
- **列**：基因（以基因名作为列标题）
- **格式**：数值型表达值

示例：
```python
import pandas as pd

# 加载表达矩阵，基因作为列
expression_matrix = pd.read_csv('expression_data.tsv', sep='\t')
# 列：['gene1', 'gene2', 'gene3', ...]
# 行：观测数据
```

### NumPy 数组
- **形状**：（观测值数量, 基因数量）
- **要求**：需单独提供与列顺序匹配的基因名称列表

示例：
```python
import numpy as np

expression_matrix = np.genfromtxt('expression_data.tsv', delimiter='\t', skip_header=1)
with open('expression_data.tsv') as f:
    gene_names = [gene.strip() for gene in f.readline().split('\t')]

assert expression_matrix.shape[1] == len(gene_names)
```

## 转录因子 (TFs)

可选地提供转录因子名称列表以限制调控推断范围：

```python
from arboreto.utils import load_tf_names

# 从文件加载（每行一个转录因子）
tf_names = load_tf_names('transcription_factors.txt')

# 或直接定义
tf_names = ['TF1', 'TF2', 'TF3']
```

如未提供，则所有基因均被视为潜在调控因子。

## 基础推断工作流程

### 使用 Pandas 数据框

```python
import pandas as pd
from arboreto.utils import load_tf_names
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 加载表达数据
    expression_matrix = pd.read_csv('expression_data.tsv', sep='\t')

    # 加载转录因子（可选）
    tf_names = load_tf_names('tf_list.txt')

    # 运行 GRN 推断
    network = grnboost2(
        expression_data=expression_matrix,
        tf_names=tf_names  # 可选
    )

    # 保存结果
    network.to_csv('network_output.tsv', sep='\t', index=False, header=False)
```

**关键**：必须使用 `if __name__ == '__main__':` 保护，因为 Dask 在内部会生成新进程。

### 使用 NumPy 数组

```python
import numpy as np
from arboreto.algo import grnboost2

if __name__ == '__main__':
    # 加载表达矩阵
    expression_matrix = np.genfromtxt('expression_data.tsv', delimiter='\t', skip_header=1)

    # 从标题行提取基因名称
    with open('expression_data.tsv') as f:
        gene_names = [gene.strip() for gene in f.readline().split('\t')]

    # 验证维度匹配
    assert expression_matrix.shape[1] == len(gene_names)

    # 使用显式基因名称运行推断
    network = grnboost2(
        expression_data=expression_matrix,
        gene_names=gene_names,
        tf_names=tf_names
    )

    network.to_csv('network_output.tsv', sep='\t', index=False, header=False)
```

## 输出格式

Arboreto 返回一个包含三列的 Pandas 数据框：

| 列名 | 描述 |
|------|------|
| `TF` | 转录因子（调控因子）基因名称 |
| `target` | 目标基因名称 |
| `importance` | 调控重要性分数（值越高表示调控作用越强） |

示例输出：
```
TF1    gene5    0.856
TF2    gene12   0.743
TF1    gene8    0.621
```

## 设置随机种子

为获得可重现的结果，请提供种子参数：

```python
network = grnboost2(
    expression_data=expression_matrix,
    tf_names=tf_names,
    seed=777
)
```

## 算法选择

在大多数情况下使用 `grnboost2()`（速度更快，可处理大型数据集）：
```python
from arboreto.algo import grnboost2
network = grnboost2(expression_data=expression_matrix)
```

使用 `genie3()` 进行比较或满足特定需求：
```python
from arboreto.algo import genie3
network = genie3(expression_data=expression_matrix)
```

详细算法比较请参见 `references/algorithms.md`。
