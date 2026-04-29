# Datamol 描述符与可视化参考

## 描述符模块 (`datamol.descriptors`)

描述符模块提供计算分子属性和描述符的工具。

### 专用描述符函数

#### `dm.descriptors.n_aromatic_atoms(mol)`
计算芳香原子数量。
- **返回**：整数计数
- **使用场景**：芳香性分析

#### `dm.descriptors.n_aromatic_atoms_proportion(mol)`
计算芳香原子与总重原子比例。
- **返回**：0到1之间的浮点数
- **使用场景**：量化芳香特性

#### `dm.descriptors.n_charged_atoms(mol)`
统计带非零形式电荷的原子数。
- **返回**：整数计数
- **使用场景**：电荷分布分析

#### `dm.descriptors.n_rigid_bonds(mol)`
统计非旋转键数量（非单键且非环键）。
- **返回**：整数计数
- **使用场景**：分子柔性评估

#### `dm.descriptors.n_stereo_centers(mol)`
统计立体中心（手性中心）数量。
- **返回**：整数计数
- **使用场景**：立体化学分析

#### `dm.descriptors.n_stereo_centers_unspecified(mol)`
统计未指定立体构型的立体中心数量。
- **返回**：整数计数
- **使用场景**：识别不完整立体化学信息

### 批量描述符计算

#### `dm.descriptors.compute_many_descriptors(mol, properties_fn=None, add_properties=True)`
计算单个分子的多个分子属性。
- **参数**：
  - `properties_fn`：自定义描述符函数列表
  - `add_properties`：是否包含额外计算属性
- **返回**：描述符名称→值的字典
- **默认描述符包括**：
  - 分子量、LogP、氢键供体/受体数量
  - 芳香原子、立体中心、可旋转键
  - TPSA（拓扑极性表面积）
  - 环数量、杂原子数量
- **示例**：
  ```python
  mol = dm.to_mol("CCO")
  descriptors = dm.descriptors.compute_many_descriptors(mol)
  # 返回: {'mw': 46.07, 'logp': -0.03, 'hbd': 1, 'hba': 1, ...}
  ```

#### `dm.descriptors.batch_compute_many_descriptors(mols, properties_fn=None, add_properties=True, n_jobs=1, batch_size=None, progress=False)`
并行计算多个分子的描述符。
- **参数**：
  - `mols`：分子列表
  - `n_jobs`：并行任务数（-1表示使用所有核心）
  - `batch_size`：并行处理的批次大小
  - `progress`：是否显示进度条
- **返回**：每行对应一个分子的Pandas DataFrame
- **示例**：
  ```python
  mols = [dm.to_mol(smi) for smi in smiles_list]
  df = dm.descriptors.batch_compute_many_descriptors(
      mols,
      n_jobs=-1,
      progress=True
  )
  ```

### RDKit描述符访问

#### `dm.descriptors.any_rdkit_descriptor(name)`
通过名称获取RDKit中的任意描述符函数。
- **参数**：`name` - 描述符函数名（如'MolWt'、'TPSA'）
- **返回**：RDKit描述符函数
- **可用描述符**：来自`rdkit.Chem.Descriptors`和`rdkit.Chem.rdMolDescriptors`
- **示例**：
  ```python
  tpsa_fn = dm.descriptors.any_rdkit_descriptor('TPSA')
  tpsa_value = tpsa_fn(mol)
  ```

### 常见用例

**类药性过滤（Lipinski五规则）**：
```python
descriptors = dm.descriptors.compute_many_descriptors(mol)
is_druglike = (
    descriptors['mw'] <= 500 and
    descriptors['logp'] <= 5 and
    descriptors['hbd'] <= 5 and
    descriptors['hba'] <= 10
)
```

**ADME属性分析**：
```python
df = dm.descriptors.batch_compute_many_descriptors(compound_library)
# 通过TPSA筛选血脑屏障穿透性
bbb_candidates = df[df['tpsa'] < 90]
```

---

## 可视化模块 (`datamol.viz`)

可视化模块提供将分子和构象体渲染为图像的工具。

### 主可视化函数

#### `dm.viz.to_image(mols, legends=None, n_cols=4, use_svg=False, mol_size=(200, 200), highlight_atom=None, highlight_bond=None, outfile=None, max_mols=None, copy=True, indices=False, ...)`
生成分子图像网格。
- **参数**：
  - `mols`：单个分子或分子列表
  - `legends`：字符串或字符串列表作为标签（每个分子一个）
  - `n_cols`：每行分子数（默认：4）
  - `use_svg`：输出SVG格式（True）或PNG（False，默认）
  - `mol_size`：元组（宽,高）或正方形尺寸的整数
  - `highlight_atom`：需高亮的原子索引（列表或字典）
  - `highlight_bond`：需高亮的键索引（列表或字典）
  - `outfile`：保存路径（支持本地/远程，兼容fsspec）
  - `max_mols`：最大显示分子数
  - `indices`：在结构上绘制原子索引（默认：False）
  - `align`：使用MCS（最大公共子结构）对齐分子
- **返回**：图像对象（可在Jupyter中显示）或保存至文件
- **示例**：
  ```python
  # 基础网格
  dm.viz.to_image(mols[:10], legends=[dm.to_smiles(m) for m in mols[:10]])

  # 保存文件
  dm.viz.to_image(mols, outfile="molecules.png", n_cols=5)

  # 高亮子结构
  dm.viz.to_image(mol, highlight_atom=[0, 1, 2], highlight_bond=[0, 1])

  # 对齐可视化
  dm.viz.to_image(mols, align=True, legends=activity_labels)
  ```

### 构象体可视化

#### `dm.viz.conformers(mol, n_confs=None, align_conf=True, n_cols=3, sync_views=True, remove_hs=True, ...)`
在网格布局中展示多个构象体。
- **参数**：
  - `mol`：包含嵌入构象体的分子
  - `n_confs`：要显示的构象体数量或索引列表（None表示全部）
  - `align_conf`：对齐构象体以便比较（默认：True）
  - `n_cols`：网格列数（默认：3）
  - `sync_views`：交互模式下同步3D视图（默认：True）
  - `remove_hs`：为清晰度移除氢原子（默认：True）
- **返回**：构象体可视化网格
- **使用场景**：比较构象多样性
- **示例**：
  ```python
  mol_3d = dm.conformers.generate(mol, n_confs=20)
  dm.viz.conformers(mol_3d, n_confs=10, align_conf=True)
  ```

### 环形网格可视化

#### `dm.viz.circle_grid(center_mol, circle_mols, mol_size=200, circle_margin=50, act_mapper=None, ...)`
创建以中心分子为核心的同心环可视化。
- **参数**：
  - `center_mol`：中心位置分子
  - `circle_mols`：分子列表的列表（每环一个列表）
  - `mol_size`：单个分子图像尺寸
  - `circle_margin`：环间距（默认：50）
  - `act_mapper`：用于颜色编码的活性映射字典
- **返回**：环形网格图像
- **使用场景**：可视化分子邻域、SAR分析、相似性网络
- **示例**：
  ```python
  # 展示参考分子及其相似化合物
  dm.viz.circle_grid(
      center_mol=reference,
      circle_mols=[nearest_neighbors, second_tier]
  )
  ```

### 可视化最佳实践

1. **使用标签增强可读性**：始终用SMILES、ID或活性值标记分子
2. **对齐相关分子**：SAR分析时在`to_image()`中设置`align=True`
3. **调整网格尺寸**：根据分子数量和显示宽度设置`n_cols`
4. **出版物使用SVG格式**：设置`use_svg=True`获取可缩放矢量图
5. **高亮子结构**：使用`highlight_atom`和`highlight_bond`强调特征
6. **保存大型网格**：使用`outfile`参数保存而非内存显示
