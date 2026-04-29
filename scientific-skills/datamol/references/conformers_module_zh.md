# Datamol 构象模块参考

`datamol.conformers` 模块提供生成和分析3D分子构象的工具。

## 构象生成

### `dm.conformers.generate(mol, n_confs=None, rms_cutoff=None, minimize_energy=True, method='ETKDGv3', add_hs=True, ...)`
生成3D分子构象。
- **参数**：
  - `mol`: 输入分子
  - `n_confs`: 生成构象数量（若为None则基于可旋转键自动确定）
  - `rms_cutoff`: 构象去重过滤的RMS阈值（单位：埃）
  - `minimize_energy`: 应用UFF能量最小化（默认：True）
  - `method`: 嵌入方法 - 可选值：
    - `'ETDG'` - 实验性扭转距离几何
    - `'ETKDG'` - 带基础知识的ETDG
    - `'ETKDGv2'` - 增强版v2
    - `'ETKDGv3'` - 增强版v3（默认推荐）
  - `add_hs`: 嵌入前添加氢原子（默认：True，对质量至关重要）
  - `random_seed`: 设置随机种子保证可复现性
- **返回**：包含嵌入构象的分子对象
- **示例**：
  ```python
  mol = dm.to_mol("CCO")
  mol_3d = dm.conformers.generate(mol, n_confs=10, rms_cutoff=0.5)
  conformers = mol_3d.GetConformers()  # 访问所有构象
  ```

## 构象聚类

### `dm.conformers.cluster(mol, rms_cutoff=1.0, already_aligned=False, centroids=False)`
基于RMS距离进行构象分组。
- **参数**：
  - `rms_cutoff`: 聚类阈值（单位：埃，默认：1.0）
  - `already_aligned`: 构象是否已预对齐
  - `centroids`: 返回中心构象（True）或聚类分组（False）
- **返回**：聚类信息或中心构象
- **应用场景**：识别不同构象家族

### `dm.conformers.return_centroids(mol, conf_clusters, centroids=True)`
从聚类中提取代表性构象。
- **参数**：
  - `conf_clusters`: 来自`cluster()`的聚类索引序列
  - `centroids`: 返回单个分子（True）或分子列表（False）
- **返回**：中心构象（或列表）

## 构象分析

### `dm.conformers.rmsd(mol)`
计算所有构象间的两两RMSD矩阵。
- **要求**：至少2个构象
- **返回**：N×N维RMSD值矩阵
- **应用场景**：量化构象多样性

### `dm.conformers.sasa(mol, n_jobs=1, ...)`
使用FreeSASA计算溶剂可及表面积（SASA）。
- **参数**：
  - `n_jobs`: 多构象并行计算数
- **返回**：SASA值数组（每个构象对应一个值）
- **存储**：值以`'rdkit_free_sasa'`属性形式存储于各构象
- **示例**：
  ```python
  sasa_values = dm.conformers.sasa(mol_3d)
  # 或从构象属性访问
  conf = mol_3d.GetConformer(0)
  sasa = conf.GetDoubleProp('rdkit_free_sasa')
  ```

## 底层构象操作

### `dm.conformers.center_of_mass(mol, conf_id=-1, use_atoms=True, round_coord=None)`
计算分子质心。
- **参数**：
  - `conf_id`: 构象索引（-1表示首个构象）
  - `use_atoms`: 使用原子质量（True）或几何中心（False）
  - `round_coord`: 坐标舍入精度
- **返回**：质心的3D坐标
- **应用场景**：分子居中处理（用于可视化或对齐）

### `dm.conformers.get_coords(mol, conf_id=-1)`
获取构象的原子坐标。
- **返回**：原子位置的N×3维numpy数组
- **示例**：
  ```python
  positions = dm.conformers.get_coords(mol_3d, conf_id=0)
  # positions.shape: (原子数, 3)
  ```

### `dm.conformers.translate(mol, conf_id=-1, transform_matrix=None)`
通过变换矩阵重定位构象。
- **修改方式**：原位操作
- **应用场景**：分子对齐或重定位

## 工作流示例

```python
import datamol as dm

# 1. 创建分子并生成构象
mol = dm.to_mol("CC(C)CCO")  # 异戊醇
mol_3d = dm.conformers.generate(
    mol,
    n_confs=50,           # 生成50个初始构象
    rms_cutoff=0.5,       # 过滤相似构象
    minimize_energy=True   # 能量最小化
)

# 2. 分析构象
n_conformers = mol_3d.GetNumConformers()
print(f"生成 {n_conformers} 个唯一构象")

# 3. 计算SASA
sasa_values = dm.conformers.sasa(mol_3d)

# 4. 构象聚类
clusters = dm.conformers.cluster(mol_3d, rms_cutoff=1.0, centroids=False)

# 5. 获取代表性构象
centroids = dm.conformers.return_centroids(mol_3d, clusters)

# 6. 访问3D坐标
coords = dm.conformers.get_coords(mol_3d, conf_id=0)
```

## 核心概念

- **距离几何**：基于连接信息生成3D结构的方法
- **ETKDG**：采用实验性扭转角偏好和额外化学知识
- **RMS阈值**：值越低构象越独特；值越高构象越少但差异越大
- **能量最小化**：将结构松弛至最近局部能量最小值
- **氢原子**：对精确3D几何结构至关重要 - 嵌入时务必包含
