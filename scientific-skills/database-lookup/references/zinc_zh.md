# ZINC 数据库 API

## 基础 URL

```
https://zinc.docking.org
```

## 认证

无需 API 密钥。完全开放的公共 API。

## URL 模式

资源遵循统一模式，文件扩展名指定格式：

```
/{resource}.{format}
/{resource}/{id}.{format}
/{resource}/subsets/{subset}.{format}
```

支持格式：`.json`, `.csv`, `.txt`, `.smi`, `.sdf`, `.mol2`, `.xml`, `.png`

字段选择（仅返回特定字段）：
```
/{resource}.json:field1+field2+field3
```

## 核心端点

### 通过 ZINC ID 查询物质
```
GET /substances/ZINC000000000053.json
```

### 按名称搜索
```
GET /substances.json?preferred_name=aspirin
```

### 通过 InChIKey 搜索
```
GET /substances.json?inchikey=BSYNRYMUTXBXSQ-UHFFFAOYSA-N
```

### 按分子式搜索
```
GET /substances.json?mol_formula=C9H8O4
```

### 子结构搜索（SMILES）
```
GET /substances.json?sub_id-matches=c1ccccc1&count=10
```

### 子结构搜索（SMARTS）
```
GET /substances.json?sub_id-matches-sma=[ND1]&count=10
```

### 相似性搜索（Tanimoto，ECFP4 指纹）

阈值（如 40 表示 40%）是参数名的一部分。值可为 SMILES 或 ZINC ID 编号。
```
GET /substances/?ecfp4_fp-tanimoto-40=c1ccccc1O
GET /substances/?ecfp4_fp-tanimoto-70=ZINC000000000053
```

### 浏览子集

按可购买性、药物状态、反应性或来源筛选：
```
GET /substances/subsets/fda.json              # FDA 批准药物
GET /substances/subsets/in-stock.json         # 现货化合物
GET /substances/subsets/metabolites.json      # 代谢物
GET /substances/subsets/fda+in-stock.json     # 用 + 组合子集
```

关键子集：
- **可购买性**：`in-stock`, `on-demand`, `for-sale`, `bb`（构建块）
- **药物状态**：`fda`, `world`, `in-trials`, `in-man`, `in-vivo`, `in-vitro`
- **来源**：`biogenic`, `metabolites`, `natural-products`, `endogenous`
- **反应性**：`anodyne`, `clean`, `standard`, `reactive`

### 针对基因靶点的物质
```
GET /genes/ACHE/substances.json?count=10
```

### 供应商目录
```
GET /catalogs.json                            # 列出所有供应商目录
GET /catalogs/cmcd/substances.json            # 目录中的物质
```

### 2D 结构图像（300x300 PNG）
```
GET /substances/ZINC000000000053.png
```

### 分子格式转换
```
GET /apps/mol/convert?from=CC(=O)Oc1ccccc1C(=O)O&to=inchikey
```
以纯文本返回 InChIKey。支持 SMILES、InChI 和 InChIKey 间的转换。

### 批量解析（POST）

同时解析多个名称、ZINC ID 或 SMILES：
```
POST /substances/resolved/
Content-Type: application/x-www-form-urlencoded

paste=aspirin%0Aibuprofen%0AZINC000000000053&identifiers=y&structures=y&names=y&output_format=json
```

## 查询参数

### 分页
- `count=N` — 每页结果数（大数据集慎用 `count=all`）
- `page=N` — 页码（从 1 开始）

### 排序
- `sort=mwt` — 按字段升序
- `sort=-mwt` — 降序（前缀 `-`）
- `sort=no` — 禁用排序以加速批量查询

### 属性筛选器（比较运算符）
- `mwt-le=500` — 分子量 <= 500
- `logp-ge=2` — LogP >= 2
- `hbd-le=5` — 氢键供体 <= 5
- 运算符：`-le` (<=), `-ge` (>=), `-lt` (<), `-gt` (>), `-eq` (=)

### 可搜索的物质属性

分子属性：`mwt`, `logp`, `hba`, `hbd`, `tpsa`, `rb`（可旋转键）, `num_rings`, `num_aromatic_rings`, `num_heavy_atoms`, `num_chiral_centers`, `fractioncsp3`

标识符：`zinc_id`, `smiles`, `inchikey`, `mol_formula`, `preferred_name`, `cas_numbers`

状态：`purchasable`, `reactive`, `bb`（构建块）

## 调用示例

### 获取化合物属性
```
GET /substances/ZINC000000000053.json:zinc_id+smiles+mwt+logp+hba+hbd+tpsa+mol_formula+preferred_name
```

### 按分子量排序的 FDA 药物
```
GET /substances/subsets/fda.json:zinc_id+preferred_name+mwt?sort=mwt&count=10
```

### 类药化合物（Lipinski 规则筛选）
```
GET /substances/subsets/for-sale.json?mwt-le=500&logp-le=5&hbd-le=5&hba-le=10&count=20
```

### 查找靶向特定基因的化合物
```
GET /genes/EGFR/substances.json:zinc_id+preferred_name+smiles?count=10
```

## 响应格式

```json
[
  {
    "zinc_id": "ZINC000000000053",
    "smiles": "CC(=O)Oc1ccccc1C(=O)O",
    "preferred_name": "aspirin",
    "mwt": 180.159,
    "logp": 1.31,
    "hba": 3,
    "hbd": 1,
    "tpsa": 63,
    "mol_formula": "C9H8O4",
    "inchikey": "BSYNRYMUTXBXSQ-UHFFFAOYSA-N",
    "purchasable": 5
  }
]
```

响应为 JSON 数组。单记录查询（通过 ZINC ID）返回 JSON 对象。

## 速率限制

无文档化速率限制。该 API 由公共资金支持（NIH NIGMS GM71896）。请合理使用：
- 使用 `count=` 限制结果规模
- 使用 `sort=no` 加速批量查询
- 相似性和子结构搜索计算密集——响应较慢
- 避免在大结果集上使用 `count=all`

## 特别说明

- ZINC 包含 **20 亿+** 市售化合物——务必使用 `count=` 限制结果
- ZINC ID 格式为 `ZINC000000000053`（"ZINC"后接15位零填充数字）
- `.smi` 格式返回 SMILES 字符串，适用于化学信息学流程
- `.sdf` 格式返回适用于对接软件的 3D 结构
- 子集可通过 `+` 组合（如 `fda+in-stock` = FDA批准且现货）
- 虚拟筛选工作流中，使用分箱（`/tranches/`）按分子量和 LogP 分区
