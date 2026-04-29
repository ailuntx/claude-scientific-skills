# PyDESeq2 API 参考文档

本文档提供 PyDESeq2 类、方法及工具函数的完整 API 参考。

## 核心类

### DeseqDataSet

用于差异表达分析的主类，处理从标准化到对数倍变化拟合的数据处理流程。

**用途：** 为 RNA-seq 计数数据实现离散度和对数倍变化（LFC）估计。

**初始化参数：**
- `counts`：形状为（样本×基因）的 pandas DataFrame，包含非负整数读段计数
- `metadata`：形状为（样本×变量）的 pandas DataFrame，包含样本注释
- `design`：字符串，指定统计模型的 Wilkinson 公式（例如 "~condition"、"~group + condition"）
- `refit_cooks`：布尔值，是否在移除 Cook 距离离群值后重新拟合参数（默认：True）
- `n_cpus`：整数，用于并行处理的 CPU 数量（可选）
- `quiet`：布尔值，是否抑制进度消息（默认：False）

**关键方法：**

#### `deseq2()`
运行完整的 DESeq2 流程进行标准化和离散度/LFC 拟合。

**执行步骤：**
1. 计算标准化因子（尺寸因子）
2. 拟合基因特异性离散度
3. 拟合离散度趋势曲线
4. 计算离散度先验
5. 拟合 MAP（最大后验）离散度
6. 拟合对数倍变化
7. 计算 Cook 距离用于离群值检测
8. 若 `refit_cooks=True` 则进行可选重拟合

**返回：** 无（原地修改对象）

#### `to_picklable_anndata()`
将 DeseqDataSet 转换为可通过 pickle 保存的 AnnData 对象。

**返回：** 包含以下内容的 AnnData 对象：
- `X`：计数数据矩阵
- `obs`：样本级元数据（一维）
- `var`：基因级元数据（一维）
- `varm`：基因级多维数据（如 LFC 估计值）

**用法：**
```python
import pickle
with open("result_adata.pkl", "wb") as f:
    pickle.dump(dds.to_picklable_anndata(), f)
```

**运行 deseq2() 后的属性：**
- `layers`：包含各类矩阵的字典（标准化计数等）
- `varm`：包含基因级结果的字典（对数倍变化、离散度等）
- `obsm`：包含样本级信息的字典
- `uns`：包含全局参数的字典

---

### DeseqStats

用于执行统计检验和计算差异表达 p 值的类。

**用途：** 通过 Wald 检验和可选 LFC 收缩实现 PyDESeq2 统计检验。

**初始化参数：**
- `dds`：已通过 `deseq2()` 处理的 DeseqDataSet 对象
- `contrast`：列表或 None，指定检验的对比组
  - 格式：`[变量, 检验水平, 参考水平]`
  - 示例：`["condition", "treated", "control"]` 检验处理组 vs 对照组
  - 若为 None，则使用设计公式中最后一个系数
- `alpha`：浮点数，独立过滤的显著性阈值（默认：0.05）
- `cooks_filter`：布尔值，是否基于 Cook 距离过滤离群值（默认：True）
- `independent_filter`：布尔值，是否执行独立过滤（默认：True）
- `n_cpus`：整数，用于并行处理的 CPU 数量（可选）
- `quiet`：布尔值，是否抑制进度消息（默认：False）

**关键方法：**

#### `summary()`
执行 Wald 检验并计算 p 值及校正 p 值。

**执行步骤：**
1. 为指定对比组运行 Wald 统计检验
2. 可选的 Cook 距离过滤
3. 可选的独立过滤以移除低功效检验
4. 多重检验校正（Benjamini-Hochberg 过程）

**返回：** 无（结果存储在 `results_df` 属性中）

**结果数据框列：**
- `baseMean`：所有样本的平均标准化计数
- `log2FoldChange`：条件间的 log2 倍变化
- `lfcSE`：log2 倍变化的标准误
- `stat`：Wald 检验统计量
- `pvalue`：原始 p 值
- `padj`：校正 p 值（FDR 校正后）

#### `lfc_shrink(coeff=None)`
使用 apeGLM 方法对对数倍变化进行收缩处理。

**用途：** 降低 LFC 估计的噪声，便于可视化和排序，尤其适用于低计数或高变异性的基因。

**参数：**
- `coeff`：字符串或 None，指定收缩的系数名称（若为 None 则使用对比组的系数）

**重要提示：** 收缩仅用于可视化/排序目的。统计检验结果（p 值、校正 p 值）保持不变。

**返回：** 无（在 `results_df` 中更新收缩后的 LFC）

**属性：**
- `results_df`：包含检验结果的 pandas 数据框（执行 `summary()` 后可用）

---

## 工具函数

### `pydeseq2.utils.load_example_data(modality="single-factor")`

加载用于测试和教程的合成示例数据集。

**参数：**
- `modality`：字符串，可选 "single-factor" 或 "multi-factor"

**返回：** 元组 (counts_df, metadata_df)
- `counts_df`：包含合成计数数据的 pandas 数据框
- `metadata_df`：包含样本注释的 pandas 数据框

---

## 预处理模块

`pydeseq2.preprocessing` 模块提供数据准备工具函数。

**常用操作：**
- 基于最小读段计数的基因过滤
- 基于元数据标准的样本过滤
- 数据转换与标准化

---

## 推断类

### Inference
定义 DESeq2 相关推断方法接口的抽象基类。

### DefaultInference
使用 scipy、sklearn 和 numpy 的推断方法默认实现。

**用途：** 提供以下数学实现：
- GLM（广义线性模型）拟合
- 离散度估计
- 趋势曲线拟合
- 统计检验

---

## 数据结构要求

### 计数矩阵
- **形状：**（样本×基因）
- **类型：** pandas 数据框
- **数值：** 非负整数（原始读段计数）
- **索引：** 样本标识符（需与元数据索引匹配）
- **列名：** 基因标识符

### 元数据
- **形状：**（样本×变量）
- **类型：** pandas 数据框
- **索引：** 样本标识符（需与计数矩阵索引匹配）
- **列名：** 实验因子（如 "condition"、"batch"、"group"）
- **数值：** 设计公式中使用的分类或连续变量

### 重要说明
- 计数矩阵与元数据的样本顺序必须一致
- 元数据中的缺失值应在分析前处理
- 基因名称应保持唯一
- 计数文件通常需要转置：`counts_df = counts_df.T`

---

## 标准工作流模式

```python
from pydeseq2.dds import DeseqDataSet
from pydeseq2.ds import DeseqStats

# 1. 初始化数据集
dds = DeseqDataSet(
    counts=counts_df,
    metadata=metadata,
    design="~condition",
    refit_cooks=True
)

# 2. 拟合离散度和 LFC
dds.deseq2()

# 3. 执行统计检验
ds = DeseqStats(
    dds,
    contrast=["condition", "treated", "control"],
    alpha=0.05
)
ds.summary()

# 4. 可选：为可视化收缩 LFC
ds.lfc_shrink()

# 5. 获取结果
results = ds.results_df
```

---

## 版本兼容性

PyDESeq2 旨在匹配 DESeq2 v1.34.0 的默认设置。由于是 Python 的从零重实现，可能存在部分差异。

**测试环境：**
- Python 3.10-3.11
- anndata 0.8.0+
- numpy 1.23.0+
- pandas 1.4.3+
- scikit-learn 1.1.1+
- scipy 1.11.0+
