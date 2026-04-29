# Datamol 核心 API 参考

本文档涵盖 datamol 命名空间中的主要函数。

## 分子创建与转换

### `to_mol(mol, ...)`
将 SMILES 字符串或其他分子表示形式转换为 RDKit 分子对象。
- **参数**：接受 SMILES 字符串、InChI 或其他分子格式
- **返回**：`rdkit.Chem.Mol` 对象
- **常用用法**：`mol = dm.to_mol("CCO")`

### `from_inchi(inchi)`
将 InChI 字符串转换为分子对象。

### `from_smarts(smarts)`
将 SMARTS 模式转换为分子对象。

### `from_selfies(selfies)`
将 SELFIES 字符串转换为分子对象。

### `copy_mol(mol)`
创建分子对象的副本以避免修改原始对象。

## 分子导出

### `to_smiles(mol, ...)`
将分子对象转换为 SMILES 字符串。
- **常用参数**：`canonical=True`（规范形式）, `isomeric=True`（保留立体异构）

### `to_inchi(mol, ...)`
将分子转换为 InChI 字符串表示。

### `to_inchikey(mol)`
将分子转换为 InChI 密钥（定长哈希值）。

### `to_smarts(mol)`
将分子转换为 SMARTS 模式。

### `to_selfies(mol)`
将分子转换为 SELFIES（自引用嵌入字符串）格式。

## 净化与标准化

### `sanitize_mol(mol, ...)`
通过 mol→SMILES→mol 转换和芳香氮修复增强 RDKit 的净化操作。
- **目的**：修复常见分子结构问题
- **返回**：净化后的分子，若失败则返回 None

### `standardize_mol(mol, disconnect_metals=False, normalize=True, reionize=True, ...)`
应用综合标准化流程，包括：
- 金属键断开
- 归一化（电荷校正）
- 重电离
- 片段处理（选择最大片段）

### `standardize_smiles(smiles, ...)`
直接对 SMILES 字符串应用标准化流程。

### `fix_mol(mol)`
自动尝试修复分子结构问题。

### `fix_valence(mol)`
修正分子结构中的化合价错误。

## 分子属性

### `reorder_atoms(mol, ...)`
确保相同分子的原子排序一致性（不受原始 SMILES 表示影响）。
- **目的**：保持可复现的特征生成

### `remove_hs(mol, ...)`
从分子结构中移除氢原子。

### `add_hs(mol, ...)`
向分子结构添加显式氢原子。

## 指纹与相似性

### `to_fp(mol, fp_type='ecfp', ...)`
生成用于相似性计算的分子指纹。
- **指纹类型**：
  - `'ecfp'` - 扩展连通性指纹（Morgan）
  - `'fcfp'` - 功能连通性指纹
  - `'maccs'` - MACCS 密钥
  - `'topological'` - 拓扑指纹
  - `'atompair'` - 原子对指纹
- **常用参数**：`n_bits`（位数）, `radius`（半径）
- **返回**：Numpy 数组或 RDKit 指纹对象

### `pdist(mols, ...)`
计算分子列表中所有分子间的成对 Tanimoto 距离。
- **支持**：通过 `n_jobs` 参数实现并行处理
- **返回**：距离矩阵

### `cdist(mols1, mols2, ...)`
计算两组分子间的 Tanimoto 距离。

## 聚类与多样性

### `cluster_mols(mols, cutoff=0.2, feature_fn=None, n_jobs=1)`
使用 Butina 聚类算法对分子进行聚类。
- **参数**：
  - `cutoff`：距离阈值（默认 0.2）
  - `feature_fn`：自定义分子特征函数
  - `n_jobs`：并行数（-1 表示使用所有核心）
- **重要提示**：构建完整距离矩阵 - 适用于约 1000 个结构，不适用于 10,000+ 规模
- **返回**：聚类列表（每个聚类为分子索引列表）

### `pick_diverse(mols, npick, ...)`
基于指纹多样性选取分子多样性子集。

### `pick_centroids(mols, npick, ...)`
选取代表聚类的质心分子。

## 图操作

### `to_graph(mol)`
将分子转换为图表示以进行基于图的分析。

### `get_all_path_between(mol, start, end)`
查找分子结构中两个原子间的所有路径。

## DataFrame 集成

### `to_df(mols, smiles_column='smiles', mol_column='mol')`
将分子列表转换为 pandas DataFrame。

### `from_df(df, smiles_column='smiles', mol_column='mol')`
将 pandas DataFrame 转换为分子列表。
