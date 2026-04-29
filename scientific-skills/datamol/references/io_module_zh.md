# Datamol I/O 模块参考

`datamol.io` 模块提供跨多种格式的分子数据综合文件处理功能。

## 读取分子文件

### `dm.read_sdf(filename, sanitize=True, remove_hs=True, as_df=True, mol_column='mol', ...)`
读取结构数据文件（SDF）格式。
- **参数**：
  - `filename`：SDF文件路径（通过fsspec支持本地和远程路径）
  - `sanitize`：对分子执行净化处理
  - `remove_hs`：移除显式氢原子
  - `as_df`：返回DataFrame（True）或分子列表（False）
  - `mol_column`：DataFrame中分子列的名称
  - `n_jobs`：启用并行处理
- **返回**：DataFrame或分子列表
- **示例**：`df = dm.read_sdf("compounds.sdf")`

### `dm.read_smi(filename, smiles_column='smiles', mol_column='mol', as_df=True, ...)`
读取SMILES文件（默认空格分隔）。
- **常见格式**：SMILES后接分子ID/名称
- **示例**：`df = dm.read_smi("molecules.smi")`

### `dm.read_csv(filename, smiles_column='smiles', mol_column=None, ...)`
读取CSV文件，支持可选的SMILES到分子的自动转换。
- **参数**：
  - `smiles_column`：包含SMILES字符串的列
  - `mol_column`：若指定，将从SMILES列创建分子对象
- **示例**：`df = dm.read_csv("data.csv", smiles_column="SMILES", mol_column="mol")`

### `dm.read_excel(filename, sheet_name=0, smiles_column='smiles', mol_column=None, ...)`
读取Excel文件并处理分子数据。
- **参数**：
  - `sheet_name`：读取的工作表（索引或名称）
  - 其他参数与`read_csv`类似
- **示例**：`df = dm.read_excel("compounds.xlsx", sheet_name="Sheet1")`

### `dm.read_molblock(molblock, sanitize=True, remove_hs=True)`
解析MOL块字符串（分子结构的文本表示）。

### `dm.read_mol2file(filename, sanitize=True, remove_hs=True, cleanupSubstructures=True)`
读取Mol2格式文件。

### `dm.read_pdbfile(filename, sanitize=True, remove_hs=True, proximityBonding=True)`
读取蛋白质数据库（PDB）格式文件。

### `dm.read_pdbblock(pdbblock, sanitize=True, remove_hs=True, proximityBonding=True)`
解析PDB块字符串。

### `dm.open_df(filename, ...)`
通用DataFrame读取器——自动检测格式。
- **支持格式**：CSV、Excel、Parquet、JSON、SDF
- **示例**：`df = dm.open_df("data.csv")` 或 `df = dm.open_df("molecules.sdf")`

## 写入分子文件

### `dm.to_sdf(mols, filename, mol_column=None, ...)`
将分子写入SDF文件。
- **输入类型**：
  - 分子列表
  - 包含分子列的DataFrame
  - 分子序列
- **参数**：
  - `mol_column`：输入为DataFrame时的列名
- **示例**：
  ```python
  dm.to_sdf(mols, "output.sdf")
  # 或从DataFrame写入
  dm.to_sdf(df, "output.sdf", mol_column="mol")
  ```

### `dm.to_smi(mols, filename, mol_column=None, ...)`
将分子写入SMILES文件，支持可选验证。
- **格式**：SMILES字符串，可选包含分子名称/ID

### `dm.to_xlsx(df, filename, mol_columns=None, ...)`
将DataFrame导出为Excel并渲染分子图像。
- **参数**：
  - `mol_columns`：包含需渲染为图像的分子的列
- **特殊功能**：自动在Excel单元格中将分子渲染为图像
- **示例**：`dm.to_xlsx(df, "molecules.xlsx", mol_columns=["mol"])`

### `dm.to_molblock(mol, ...)`
将分子转换为MOL块字符串。

### `dm.to_pdbblock(mol, ...)`
将分子转换为PDB块字符串。

### `dm.save_df(df, filename, ...)`
以多种格式保存DataFrame（CSV、Excel、Parquet、JSON）。

## 远程文件支持

所有I/O函数通过fsspec集成支持远程文件路径：
- **支持协议**：S3（AWS）、GCS（Google Cloud）、Azure、HTTP/HTTPS
- **示例**：
  ```python
  dm.read_sdf("s3://bucket/compounds.sdf")
  dm.read_csv("https://example.com/data.csv")
  ```

## 跨函数关键参数

- **`sanitize`**：执行分子净化（默认：True）
- **`remove_hs`**：移除显式氢原子（默认：True）
- **`as_df`**：返回DataFrame或列表（多数函数默认：True）
- **`n_jobs`**：启用并行处理（None=所有核心，1=顺序执行）
- **`mol_column`**：DataFrame中分子列的名称
- **`smiles_column`**：DataFrame中SMILES列的名称
