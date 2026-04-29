---
name: datamol
description: RDKit 的 Python 风格封装，提供简化接口和合理默认值。适用于标准药物发现任务，包括 SMILES 解析、标准化、描述符计算、指纹生成、聚类、3D 构象生成和并行处理。返回原生 rdkit.Chem.Mol 对象。如需高级控制或自定义参数，请直接使用 RDKit。
license: Apache-2.0 license
metadata:
    skill-author: K-Dense Inc.
---

# Datamol 化学信息学技能

## 概述

Datamol 是一个轻量级的 Python 库，为 RDKit 分子化学信息学提供 Python 风格的抽象层。通过合理的默认值、高效并行化和现代化 I/O 能力简化复杂分子操作。所有分子对象均为原生 `rdkit.Chem.Mol` 实例，确保与 RDKit 生态完全兼容。

**核心能力**：
- 分子格式转换（SMILES、SELFIES、InChI）
- 结构标准化与净化
- 分子描述符与指纹计算
- 3D 构象生成与分析
- 聚类与多样性筛选
- 骨架与片段分析
- 化学反应应用
- 可视化与分子对齐
- 批处理并行化
- 通过 fsspec 支持云存储

## 安装与配置

安装指南：
```bash
uv pip install datamol
```

**导入约定**：
```python
import datamol as dm
```

## 核心工作流

### 1. 基础分子处理

**从 SMILES 创建分子**：
```python
import datamol as dm

# 单个分子
mol = dm.to_mol("CCO")  # 乙醇

# 从 SMILES 列表创建
smiles_list = ["CCO", "c1ccccc1", "CC(=O)O"]
mols = [dm.to_mol(smi) for smi in smiles_list]

# 错误处理
mol = dm.to_mol("invalid_smiles")  # 返回 None
if mol is None:
    print("SMILES 解析失败")
```

**将分子转为 SMILES**：
```python
# 规范 SMILES
smiles = dm.to_smiles(mol)

# 立体化学 SMILES
smiles = dm.to_smiles(mol, isomeric=True)

# 其他格式
inchi = dm.to_inchi(mol)
inchikey = dm.to_inchikey(mol)
selfies = dm.to_selfies(mol)
```

**标准化与净化**（推荐对所有用户提供的分子执行）：
```python
# 净化分子
mol = dm.sanitize_mol(mol)

# 完整标准化（推荐用于数据集）
mol = dm.standardize_mol(
    mol,
    disconnect_metals=True,
    normalize=True,
    reionize=True
)

# 直接处理 SMILES 字符串
clean_smiles = dm.standardize_smiles(smiles)
```

### 2. 分子文件读写

完整 I/O 文档见 `references/io_module.md`。

**读取文件**：
```python
# SDF 文件（化学领域最常用）
df = dm.read_sdf("compounds.sdf", mol_column='mol')

# SMILES 文件
df = dm.read_smi("molecules.smi", smiles_column='smiles', mol_column='mol')

# 含 SMILES 列的 CSV
df = dm.read_csv("data.csv", smiles_column="SMILES", mol_column="mol")

# Excel 文件
df = dm.read_excel("compounds.xlsx", sheet_name=0, mol_column="mol")

# 通用读取器（自动检测格式）
df = dm.open_df("file.sdf")  # 支持 .sdf, .csv, .xlsx, .parquet, .json
```

**写入文件**：
```python
# 保存为 SDF
dm.to_sdf(mols, "output.sdf")
# 或从 DataFrame 保存
dm.to_sdf(df, "output.sdf", mol_column="mol")

# 保存为 SMILES 文件
dm.to_smi(mols, "output.smi")

# 带分子渲染图像的 Excel
dm.to_xlsx(df, "output.xlsx", mol_columns=["mol"])
```

**远程文件支持**（S3, GCS, HTTP）：
```python
# 从云存储读取
df = dm.read_sdf("s3://bucket/compounds.sdf")
df = dm.read_csv("https://example.com/data.csv")

# 写入云存储
dm.to_sdf(mols, "s3://bucket/output.sdf")
```

### 3. 分子描述符与性质

详细描述符文档见 `references/descriptors_viz.md`。

**计算单个分子描述符**：
```python
# 获取标准描述符集
descriptors = dm.descriptors.compute_many_descriptors(mol)
# 返回: {'mw': 46.07, 'logp': -0.03, 'hbd': 1, 'hba': 1,
#        'tpsa': 20.23, 'n_aromatic_atoms': 0, ...}
```

**批量描述符计算**（推荐用于数据集）：
```python
# 并行计算所有分子
desc_df = dm.descriptors.batch_compute_many_descriptors(
    mols,
    n_jobs=-1,      # 使用所有 CPU 核心
    progress=True   # 显示进度条
)
```

**特定描述符**：
```python
# 芳香性
n_aromatic = dm.descriptors.n_aromatic_atoms(mol)
aromatic_ratio = dm.descriptors.n_aromatic_atoms_proportion(mol)

# 立体化学
n_stereo = dm.descriptors.n_stereo_centers(mol)
n_unspec = dm.descriptors.n_stereo_centers_unspecified(mol)

# 柔性
n_rigid = dm.descriptors.n_rigid_bonds(mol)
```

**类药性过滤（Lipinski 五规则）**：
```python
# 过滤化合物
def is_druglike(mol):
    desc = dm.descriptors.compute_many_descriptors(mol)
    return (
        desc['mw'] <= 500 and
        desc['logp'] <= 5 and
        desc['hbd'] <= 5 and
        desc['hba'] <= 10
    )

druglike_mols = [mol for mol in mols if is_druglike(mol)]
```

### 4. 分子指纹与相似性

**生成指纹**：
```python
# ECFP（扩展连通性指纹，默认）
fp = dm.to_fp(mol, fp_type='ecfp', radius=2, n_bits=2048)

# 其他指纹类型
fp_maccs = dm.to_fp(mol, fp_type='maccs')
fp_topological = dm.to_fp(mol, fp_type='topological')
fp_atompair = dm.to_fp(mol, fp_type='atompair')
```

**相似性计算**：
```python
# 集合内两两距离
distance_matrix = dm.pdist(mols, n_jobs=-1)

# 两组间距离
distances = dm.cdist(query_mols, library_mols, n_jobs=-1)

# 查找最相似分子
from scipy.spatial.distance import squareform
dist_matrix = squareform(dm.pdist(mols))
# 距离越小相似度越高（Tanimoto 距离 = 1 - Tanimoto 相似度）
```

### 5. 聚类与多样性筛选

聚类细节见 `references/core_api.md`。

**Butina 聚类**：
```python
# 按结构相似性聚类分子
clusters = dm.cluster_mols(
    mols,
    cutoff=0.2,    # Tanimoto 距离阈值（0=相同，1=完全不同）
    n_jobs=-1      # 并行处理
)

# 每个聚类是分子索引列表
for i, cluster in enumerate(clusters):
    print(f"聚类 {i}: {len(cluster)} 个分子")
    cluster_mols = [mols[idx] for idx in cluster]
```

**重要提示**：Butina 聚类需构建完整距离矩阵 - 适用于约 1000 个分子，不适用于 10,000+。

**多样性筛选**：
```python
# 选取多样性子集
diverse_mols = dm.pick_diverse(
    mols,
    npick=100  # 选取 100 个多样性分子
)

# 选取聚类中心
centroids = dm.pick_centroids(
    mols,
    npick=50   # 选取 50 个代表性分子
)
```

### 6. 骨架分析

完整骨架文档见 `references/fragments_scaffolds.md`。

**提取 Murcko 骨架**：
```python
# 获取 Bemis-Murcko 骨架（核心结构）
scaffold = dm.to_scaffold_murcko(mol)
scaffold_smiles = dm.to_smiles(scaffold)
```

**基于骨架的分析**：
```python
# 按骨架分组化合物
from collections import Counter

scaffolds = [dm.to_scaffold_murcko(mol) for mol in mols]
scaffold_smiles = [dm.to_smiles(s) for s in scaffolds]

# 统计骨架频率
scaffold_counts = Counter(scaffold_smiles)
most_common = scaffold_counts.most_common(10)

# 创建骨架到分子的映射
scaffold_groups = {}
for mol, scaf_smi in zip(mols, scaffold_smiles):
    if scaf_smi not in scaffold_groups:
        scaffold_groups[scaf_smi] = []
    scaffold_groups[scaf_smi].append(mol)
```

**基于骨架的机器学习数据集划分**：
```python
# 确保训练集和测试集骨架不同
scaffold_to_mols = {}
for mol, scaf in zip(mols, scaffold_smiles):
    if scaf not in scaffold_to_mols:
        scaffold_to_mols[scaf] = []
    scaffold_to_mols[scaf].append(mol)

# 划分骨架到训练/测试集
import random
scaffolds = list(scaffold_to_mols.keys())
random.shuffle(scaffolds)
split_idx = int(0.8 * len(scaffolds))
train_scaffolds = scaffolds[:split_idx]
test_scaffolds = scaffolds[split_idx:]

# 获取各分集分子
train_mols = [mol for scaf in train_scaffolds for mol in scaffold_to_mols[scaf]]
test_mols = [mol for scaf in test_scaffolds for mol in scaffold_to_mols[scaf]]
```

### 7. 分子片段化

片段化细节见 `references/fragments_scaffolds.md`。

**BRICS 片段化**（16 种键类型）：
```python
# 片段化分子
fragments = dm.fragment.brics(mol)
# 返回: 带连接点的片段 SMILES 集合（如 '[1*]CCN'）
```

**RECAP 片段化**（11 种键类型）：
```python
fragments = dm.fragment.recap(mol)
```

**片段分析**：
```python
# 在化合物库中查找常见片段
from collections import Counter

all_fragments = []
for mol in mols:
    frags = dm.fragment.brics(mol)
    all_fragments.extend(frags)

fragment_counts = Counter(all_fragments)
common_frags = fragment_counts.most_common(20)

# 基于片段的评分
def fragment_score(mol, reference_fragments):
    mol_frags = dm.fragment.brics(mol)
    overlap = mol_frags.intersection(reference_fragments)
    return len(overlap) / len(mol_frags) if mol_frags else 0
```

### 8. 3D 构象生成

详细构象文档见 `references/conformers_module.md`。

**生成构象**：
```python
# 生成 3D 构象
mol_3d = dm.conformers.generate(
    mol,
    n_confs=50,           # 生成数量（None 时自动确定）
    rms_cutoff=0.5,       # 过滤相似构象（埃）
    minimize_energy=True,  # 使用 UFF 力场优化
    method='ETKDGv3'      # 嵌入方法（推荐）
)

# 访问构象
n_conformers = mol_3d.GetNumConformers()
conf = mol_3d.GetConformer(0)  # 获取第一个构象
positions = conf.GetPositions()  # 原子坐标的 Nx3 数组
```

**构象聚类**：
```python
# 按 RMSD 聚类构象
clusters = dm.conformers.cluster(
    mol_3d,
    rms_cutoff=1.0,
    centroids=False
)

# 获取代表性构象
centroids = dm.conformers.return_centroids(mol_3d, clusters)
```

**SASA 计算**：
```python
# 计算溶剂可及表面积
sasa_values = dm.conformers.sasa(mol_3d, n_jobs=-1)

# 从构象属性访问 SASA
conf = mol_3d.GetConformer(0)
sasa = conf.GetDoubleProp('rdkit_free_sasa')
```

### 9. 可视化

可视化文档见 `references/descriptors_viz.md`。

**基础分子网格**：
```python
# 可视化分子
dm.viz.to_image(
    mols[:20],
    legends=[dm.to_smiles(m) for m in mols[:20]],
    n_cols=5,
    mol_size=(300, 300)
)

# 保存到文件
dm.viz.to_image(mols, outfile="molecules.png")

# 出版物用 SVG
dm.viz.to_image(mols, outfile="molecules.svg", use_svg=True)
```

**对齐可视化**（用于构效关系分析）：
```python
# 通过最大公共子结构对齐分子
dm.viz.to_image(
    similar_mols,
    align=True,  # 启用 MCS 对齐
    legends=activity_labels,
    n_cols=4
)
```

**高亮子结构**：
```python
# 高亮特定原子和键
dm.viz.to_image(
    mol,
    highlight_atom=[0, 1, 2, 3],  # 原子索引
    highlight_bond=[0, 1, 2]      # 键索引
)
```

**构象可视化**：
```python
# 展示多个构象
dm.viz.conformers(
    mol_3d,
    n_confs=10,
    align_conf=True,
    n_cols=3
)
```

### 10. 化学反应

反应文档见 `references/reactions_data.md`。

**应用反应**：
```python
from rdkit.Chem import rdChemReactions

# 从 SMARTS 定义反应
rxn_smarts = '[C:1](=[O:2])[OH:3]>>[C:1](=[O:2])[Cl:3]'
rxn = rdChemReactions.ReactionFromSmarts(rxn_smarts)

# 应用于分子
reactant = dm.to_mol("CC(=O)O")  # 乙酸
product = dm.reactions.apply_reaction(
    rxn,
    (reactant,),
    sanitize=True
)

# 转为 SMILES
product_smiles = dm.to_smiles(product)
```

**批量反应应用**：
```python
# 应用于分子库
products = []
for mol in reactant_mols:
    try:
        prod = dm.reactions.apply_reaction(rxn, (mol,))
        if prod is not None:
            products.append(prod)
    except Exception as e:
        print(f"反应失败: {e}")
```

## 并行化

Datamol 内置多种操作的并行化支持。使用 `n_jobs` 参数：
- `n_jobs=1`：顺序执行（无并行）
- `n_jobs=-1`：使用所有可用 CPU 核心
- `n_jobs=4`：使用 4 个核心

**支持并行化的函数**：
- `dm.read_sdf(..., n_jobs=-1)`
- `dm.descriptors.batch_compute_many_descriptors(..., n_jobs=-1)`
- `dm.cluster_mols(..., n_jobs=-1)`
- `

```markdown
(desc_df['hbd'] <= 5) &
    (desc_df['hba'] <= 10)
)
filtered_df = df[druglike]

# 5. 聚类并选择多样化子集
diverse_mols = dm.pick_diverse(
    filtered_df['mol'].tolist(),
    npick=100
)

# 6. 可视化结果
dm.viz.to_image(
    diverse_mols,
    legends=[dm.to_smiles(m) for m in diverse_mols],
    outfile="diverse_compounds.png",
    n_cols=10
)
```

### 构效关系 (SAR) 分析

```python
# 按骨架分组
scaffolds = [dm.to_scaffold_murcko(mol) for mol in mols]
scaffold_smiles = [dm.to_smiles(s) for s in scaffolds]

# 创建含活性数据的DataFrame
sar_df = pd.DataFrame({
    'mol': mols,
    'scaffold': scaffold_smiles,
    'activity': activities  # 用户提供的活性数据
})

# 分析每个骨架系列
for scaffold, group in sar_df.groupby('scaffold'):
    if len(group) >= 3:  # 需要多个样本
        print(f"\n骨架: {scaffold}")
        print(f"数量: {len(group)}")
        print(f"活性范围: {group['activity'].min():.2f} - {group['activity'].max():.2f}")

        # 以活性值为图例可视化
        dm.viz.to_image(
            group['mol'].tolist(),
            legends=[f"活性: {act:.2f}" for act in group['activity']],
            align=True  # 按共同子结构对齐
        )
```

### 虚拟筛选流程

```python
# 1. 生成查询分子和化合物库的指纹
query_fps = [dm.to_fp(mol) for mol in query_actives]
library_fps = [dm.to_fp(mol) for mol in library_mols]

# 2. 计算相似度
from scipy.spatial.distance import cdist
import numpy as np

distances = dm.cdist(query_actives, library_mols, n_jobs=-1)

# 3. 寻找最接近匹配（到任意查询的最小距离）
min_distances = distances.min(axis=0)
similarities = 1 - min_distances  # 距离转相似度

# 4. 排序并选择最优命中化合物
top_indices = np.argsort(similarities)[::-1][:100]  # 前100名
top_hits = [library_mols[i] for i in top_indices]
top_scores = [similarities[i] for i in top_indices]

# 5. 可视化命中化合物
dm.viz.to_image(
    top_hits[:20],
    legends=[f"相似度: {score:.3f}" for score in top_scores[:20]],
    outfile="screening_hits.png"
)
```

## 参考文档

详细API文档请查阅以下参考文件：

- **`references/core_api.md`**：核心命名空间函数（转换、标准化、指纹、聚类）
- **`references/io_module.md`**：文件I/O操作（读写SDF/CSV/Excel/远程文件）
- **`references/conformers_module.md`**：3D构象生成、聚类、SASA计算
- **`references/descriptors_viz.md`**：分子描述符与可视化函数
- **`references/fragments_scaffolds.md`**：骨架提取、BRICS/RECAP片段化
- **`references/reactions_data.md`**：化学反应与示例数据集

## 最佳实践

1. **始终标准化外部来源的分子**：
   ```python
   mol = dm.standardize_mol(mol, disconnect_metals=True, normalize=True, reionize=True)
   ```

2. **分子解析后检查空值**：
   ```python
   mol = dm.to_mol(smiles)
   if mol is None:
       # 处理无效SMILES
   ```

3. **大数据集使用并行处理**：
   ```python
   result = dm.operation(..., n_jobs=-1, progress=True)
   ```

4. **利用fsspec访问云存储**：
   ```python
   df = dm.read_sdf("s3://bucket/compounds.sdf")
   ```

5. **根据场景选用合适指纹**：
   - ECFP (Morgan)：通用结构相似性
   - MACCS：快速，特征空间小
   - 原子对：考虑原子对与距离

6. **注意规模限制**：
   - Butina聚类：约1,000个分子（全距离矩阵）
   - 大数据集：使用多样性选择或分层方法

7. **机器学习中按骨架划分**：确保训练集/测试集通过骨架分离

8. **可视化SAR系列时对齐分子**

## 错误处理

```python
# 安全创建分子
def safe_to_mol(smiles):
    try:
        mol = dm.to_mol(smiles)
        if mol is not None:
            mol = dm.standardize_mol(mol)
        return mol
    except Exception as e:
        print(f"处理失败 {smiles}: {e}")
        return None

# 安全批量处理
valid_mols = []
for smiles in smiles_list:
    mol = safe_to_mol(smiles)
    if mol is not None:
        valid_mols.append(mol)
```

## 机器学习集成

```python
# 特征生成
X = np.array([dm.to_fp(mol) for mol in mols])

# 或使用描述符
desc_df = dm.descriptors.batch_compute_many_descriptors(mols, n_jobs=-1)
X = desc_df.values

# 训练模型
from sklearn.ensemble import RandomForestRegressor
model = RandomForestRegressor()
model.fit(X, y_target)

# 预测
predictions = model.predict(X_test)
```

## 故障排除

**问题**：分子解析失败
- **解决方案**：先使用`dm.standardize_smiles()`或尝试`dm.fix_mol()`

**问题**：聚类内存错误
- **解决方案**：大数据集使用`dm.pick_diverse()`替代全聚类

**问题**：构象生成缓慢
- **解决方案**：减少`n_confs`或增大`rms_cutoff`以生成更少构象

**问题**：远程文件访问失败
- **解决方案**：确保已安装fsspec及对应云服务库（s3fs, gcsfs等）

## 附加资源

- **Datamol文档**：https://docs.datamol.io/
- **RDKit文档**：https://www.rdkit.org/docs/
- **GitHub仓库**：https://github.com/datamol-io/datamol
```
