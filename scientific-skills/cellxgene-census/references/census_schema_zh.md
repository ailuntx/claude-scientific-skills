# CZ CELLxGENE 普查数据模式参考

## 概述

CZ CELLxGENE 普查是基于 TileDB-SOMA 框架构建的版本化单细胞数据集。本文档描述数据结构、可用元数据字段及查询语法。

## 高层结构

普查数据组织为 `SOMACollection`，包含两个主要组件：

### 1. census_info
摘要信息包括：
- **summary**：构建日期、细胞计数、数据集统计
- **datasets**：来自 CELLxGENE Discover 的所有数据集及其元数据
- **summary_cell_counts**：按元数据类别分层的细胞计数

### 2. census_data
物种特定的 `SOMAExperiment` 对象：
- **"homo_sapiens"**：人类单细胞数据
- **"mus_musculus"**：小鼠单细胞数据

## 各物种数据结构

每个物种实验包含：

### obs（细胞元数据）
细胞级注释，存储为 `SOMADataFrame`。访问方式：
```python
census["census_data"]["homo_sapiens"].obs
```

### ms["RNA"]（测量数据）
RNA 测量数据包括：
- **X**：数据矩阵及其分层：
  - `raw`：原始计数数据
  - `normalized`：（如可用）标准化计数
- **var**：基因元数据
- **feature_dataset_presence_matrix**：稀疏布尔数组，显示各数据集检测到的基因

## 细胞元数据字段（obs）

### 必需/核心字段

**身份与数据集：**
- `soma_joinid`：用于连接的唯一整数标识符
- `dataset_id`：源数据集标识符
- `is_primary_data`：布尔标志（True=唯一细胞，False=跨数据集重复）

**细胞类型：**
- `cell_type`：人类可读的细胞类型名称
- `cell_type_ontology_term_id`：标准化本体术语（如 "CL:0000236"）

**组织：**
- `tissue`：具体组织名称
- `tissue_general`：广义组织类别（用于分组）
- `tissue_ontology_term_id`：标准化本体术语

**检测方法：**
- `assay`：使用的测序技术
- `assay_ontology_term_id`：标准化本体术语

**疾病：**
- `disease`：疾病状态或条件
- `disease_ontology_term_id`：标准化本体术语

**供体：**
- `donor_id`：唯一供体标识符
- `sex`：生物性别（male, female, unknown）
- `self_reported_ethnicity`：种族信息
- `development_stage`：生命阶段（adult, child, embryonic 等）
- `development_stage_ontology_term_id`：标准化本体术语

**生物体：**
- `organism`：学名（Homo sapiens, Mus musculus）
- `organism_ontology_term_id`：标准化本体术语

**技术参数：**
- `suspension_type`：样本制备类型（cell, nucleus, na）

## 基因元数据字段（var）

访问方式：
```python
census["census_data"]["homo_sapiens"].ms["RNA"].var
```

**可用字段：**
- `soma_joinid`：用于连接的唯一整数标识符
- `feature_id`：Ensembl 基因 ID（如 "ENSG00000161798"）
- `feature_name`：基因符号（如 "FOXP2"）
- `feature_length`：基因长度（以碱基对计）

## 值筛选语法

查询使用类 Python 表达式进行过滤，语法由 TileDB-SOMA 处理。

### 比较运算符
- `==`：等于
- `!=`：不等于
- `<`, `>`, `<=`, `>=`：数值比较
- `in`：成员测试（如 `feature_id in ['ENSG00000161798', 'ENSG00000188229']`）

### 逻辑运算符
- `and`, `&`：逻辑与
- `or`, `|`：逻辑或

### 示例

**单条件筛选：**
```python
value_filter="cell_type == 'B cell'"
```

**多条件 AND 组合：**
```python
value_filter="cell_type == 'B cell' and tissue_general == 'lung' and is_primary_data == True"
```

**使用 IN 匹配多值：**
```python
value_filter="tissue in ['lung', 'liver', 'kidney']"
```

**复合条件：**
```python
value_filter="(cell_type == 'neuron' or cell_type == 'astrocyte') and disease != 'normal'"
```

**基因筛选：**
```python
var_value_filter="feature_name in ['CD4', 'CD8A', 'CD19']"
```

## 数据纳入标准

普查包含所有符合以下条件的 CZ CELLxGENE Discover 数据：

1. **物种**：人类（*Homo sapiens*）或小鼠（*Mus musculus*）
2. **技术**：经批准的 RNA 测序技术
3. **计数类型**：仅原始计数（不含纯处理/标准化数据）
4. **元数据**：遵循 CELLxGENE 模式标准化
5. **空间与非空间数据**：包含传统和空间转录组学

## 重要数据特征

### 重复细胞
细胞可能出现在多个数据集中。多数分析中应使用 `is_primary_data == True` 筛选唯一细胞。

### 计数类型
普查包含：
- **分子计数**：来自基于 UMI 的方法
- **全基因测序读数**：来自非 UMI 方法
这些数据可能需要不同的标准化方法。

### 版本控制
普查发布版本化（如 "2023-07-25", "stable"）。为保障分析可复现，请指定版本：
```python
census = cellxgene_census.open_soma(census_version="2023-07-25")
```

## 数据集存在矩阵

访问各数据集检测到的基因：
```python
presence_matrix = census["census_data"]["homo_sapiens"].ms["RNA"]["feature_dataset_presence_matrix"]
```

此稀疏布尔矩阵有助于理解：
- 基因在数据集间的覆盖范围
- 特定基因分析应包含的数据集
- 与基因覆盖相关的技术批次效应

## SOMA 对象类型

使用的核心 TileDB-SOMA 对象：
- **DataFrame**：表格数据（obs, var）
- **SparseNDArray**：稀疏矩阵（X 分层, 存在矩阵）
- **DenseNDArray**：密集数组（较少见）
- **Collection**：相关对象的容器
- **Experiment**：测量数据的顶层容器
- **SOMAScene**：空间转录组场景
- **obs_spatial_presence**：空间数据可用性
