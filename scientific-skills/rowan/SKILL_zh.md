---
name: rowan
description: Rowan 是一个云原生的分子建模与药物化学工作流平台，提供 Python API。用于 pKa 和 macropKa 预测、构象异构体与互变异构体系综生成、分子对接与类似物对接、蛋白质-配体共折叠、多序列比对（MSA）生成、分子动力学、渗透性计算、描述符工作流及相关小分子或蛋白质建模任务。特别适合程序化批量筛选、多步骤化学流程以及需要本地 HPC/GPU 基础设施维护的工作流。
license: 专有（需 API 密钥）
compatibility: Python 3.12+，需 API 密钥
metadata:
  skill-author: Rowan Science
  trigger-keywords: ["pKa 预测", "分子对接", "构象搜索", "化学工作流", "药物发现", "SMILES", "蛋白质结构", "批量分子建模", "云端化学"]
---

# Rowan：云原生分子建模与药物设计工作流

## 概述

Rowan 是面向分子模拟、药物化学和基于结构设计的云原生工作流平台。其 Python API 提供统一接口，支持小分子建模、性质预测、分子对接、分子动力学及 AI 结构工作流。

当您需要以编程方式运行药物化学或分子设计工作流，且不希望维护本地 HPC 基础设施、GPU 资源或分散的建模工具时，Rowan 是理想选择。平台自动处理所有基础设施、结果管理和计算扩展。

## 适用场景

**Rowan 适用于：**

- 量子化学、半经验方法或神经网络势函数
- 批量性质预测（pKa、描述符、渗透性、溶解度）
- 构象异构体与互变异构体系综生成
- 分子对接工作流（单配体、类似物系列、构象优化）
- 蛋白质-配体共折叠与多序列比对（MSA）生成
- 多步骤化学流程（如互变异构搜索 → 对接 → 构象分析）
- 需要稳定可扩展基础设施的批量药物化学项目

**Rowan 不适用于：**
- 简单分子 I/O 操作（请直接使用 RDKit）
- 后 Hartree-Fock *ab initio* 量子化学或相对论计算

## 访问与定价模式

Rowan 采用基于积分的用量模型。所有用户（包括免费层）均可创建 API 密钥并使用 Python API。

### 免费层权限

- 访问所有 Rowan 核心工作流
- 每周 20 积分
- 注册赠送 500 积分

### 定价与积分消耗

积分按计算类型消耗：

- **CPU**：每分钟 1 积分
- **GPU**：每分钟 3 积分
- **H100/H200 GPU**：每分钟 7 积分

购买的积分按单价计费，自购买日起有效期一年。

### 典型成本估算

| 工作流 | 典型耗时 | 预估积分 | 说明 |
|----------|----------------|-------------------|-------|
| 描述符计算 | <1 分钟 | 0.5–2 | 轻量级，适合初筛 |
| pKa（单质子转移） | 2–5 分钟 | 2–5 | 取决于分子大小 |
| MacropKa（pH 0–14） | 5–15 分钟 | 5–15 | 采样范围更广，成本更高 |
| 构象搜索 | 3–10 分钟 | 3–10 | 系综质量影响结果 |
| 互变异构搜索 | 2–5 分钟 | 2–5 | 杂环体系 |
| 单配体对接 | 5–20 分钟 | 5–20 | 取决于口袋大小与优化程度 |
| 类似物对接系列（10–50 个配体） | 30–120 分钟 | 30–100+ | 共享参考框架 |
| MSA 生成 | 5–30 分钟 | 5–30 | 依赖序列长度 |
| 蛋白质-配体共折叠 | 15–60 分钟 | 20–50+ | AI 结构预测，GPU 密集型 |

## 快速入门

```bash
uv pip install rowan-python
```

```python
import rowan
rowan.api_key = "your_api_key_here"  # 或设置 ROWAN_API_KEY 环境变量

# 提交描述符工作流——1 分钟内完成
wf = rowan.submit_descriptors_workflow("CC(=O)Oc1ccccc1C(=O)O", name="阿司匹林")
result = wf.result()

print(result.descriptors['MW'])    # 180.16
print(result.descriptors['SLogP']) # 1.19
print(result.descriptors['TPSA'])  # 59.44
```

若成功输出，说明环境配置正确。

## 安装

```bash
uv pip install rowan-python
# 或：pip install rowan-python
```

## 用户与 Webhook 管理

### 认证

通过环境变量设置 API 密钥（推荐）：

```bash
export ROWAN_API_KEY="your_api_key_here"
```

或在 Python 中直接设置：

```python
import rowan
rowan.api_key = "your_api_key_here"
```

验证认证状态：

```python
import rowan
user = rowan.whoami()  # 认证成功返回用户信息
print(f"用户: {user.email}")
print(f"可用积分: {user.credits_available_string}")
```

### Webhook 密钥管理

通过用户账户管理 Webhook 签名验证密钥：

```python
import rowan

# 获取当前 Webhook 密钥（无密钥时返回 None）
secret = rowan.get_webhook_secret()
if secret is None:
    secret = rowan.create_webhook_secret()
print(f"密钥: {secret.secret}")

# 轮换密钥（使旧密钥失效，创建新密钥）
# 定期执行以提升安全性
new_secret = rowan.rotate_webhook_secret()
print(f"新密钥已创建（旧密钥禁用）: {new_secret.secret}")

# 验证传入 Webhook 签名
is_valid = rowan.verify_webhook_secret(
    request_body=b"...",           # 原始请求体（字节）
    signature="X-Rowan-Signature", # 来自请求头
    secret=secret.secret
)
```

## 分子输入格式

Rowan 支持以下分子格式：

- **SMILES**（首选）：`"CCO"`、`"c1ccccc1O"`
- **SMARTS 模式**（部分工作流）：用于子结构匹配的 SMARTS 子集
- **InChI**（若 API 版本支持）：`"InChI=1S/C2H6O/c1-2-3/h3H,2H2,1H3"`

API 将验证输入，若分子无法解析则抛出 `rowan.ValidationError`。为保持可复现性，请始终使用规范化 SMILES。

**提示：** 提交前用 RDKit 验证 SMILES：

```python
from rdkit import Chem
smiles = "CCO"
mol = Chem.MolFromSmiles(smiles)
if mol is None:
    raise ValueError(f"无效 SMILES: {smiles}")
```

## 核心使用模式

多数 Rowan 任务遵循三步模式：

1. **提交**工作流
2. **等待**完成（可选流式处理）
3. 通过便捷属性**获取**类型化结果

```python
import rowan

# 1. 提交——使用特定工作流函数（非通用 submit_workflow）
workflow = rowan.submit_descriptors_workflow(
    "CC(=O)Oc1ccccc1C(=O)O",
    name="阿司匹林描述符",
)

# 2. & 3. 等待并获取结果
result = workflow.result()  # 阻塞至完成（默认：wait=True, poll_interval=5）
print(result.data)              # 原始字典
print(result.descriptors['MW']) # 180.16——使用 result.descriptors 字典而非 result.molecular_weight
```

长时任务可使用流式处理：

```python
for partial in workflow.stream_result(poll_interval=5):
    print(f"进度: {partial.complete}%")
    print(partial.data)
```

### result() 与 stream_result() 对比

| 模式 | 适用场景 | 耗时 |
|---------|----------|----------|
| `result()` | 可等待完整结果 | 通常 <5 分钟 |
| `stream_result()` | 需进度反馈或早期部分结果 | >5 分钟或交互式场景 |

**建议：** 描述符、pKa 用 `result()`；构象搜索、对接、共折叠用 `stream_result()`。

## 结果处理

Rowan API 提供**类型化工作流结果对象**及便捷属性。

### 使用类型化属性与 .data

结果有两种访问方式：

1. **便捷属性**（首选）：`result.descriptors`、`result.best_pose`、`result.conformer_energies`
2. **原始回退**：`result.data`——API 原始字典

示例：

```python
result = rowan.submit_descriptors_workflow(
    "CCO",
    name="乙醇",
).result()

# 便捷属性（返回所有描述符字典）：
print(result.descriptors['MW'])   # 46.042
print(result.descriptors['SLogP'])  # -0.001
print(result.descriptors['TPSA'])   # 57.96

# 原始数据回退（描述符嵌套在 'descriptors' 键下）：
print(result.data['descriptors'])
# {'MW': 46.042, 'SLogP': -0.001, 'TPSA': 57.96, 'nHBDon': 1.0, 'nHBAcc': 1.0, ...}
```

**注意：** `DescriptorsResult` **无** `molecular_weight` 属性。描述符键使用短名称（`MW`、`SLogP`、`nHBDon`）而非长名称。

### 缓存刷新

部分结果属性延迟加载（如构象几何结构、蛋白质结构）。刷新方式：

```python
result.clear_cache()
new_structures = result.conformer_molecules  # 重新获取
```

## 项目、文件夹与组织管理

复杂任务建议使用项目与文件夹管理。

### 项目管理

```python
import rowan

# 创建项目
project = rowan.create_project(name="CDK2 先导化合物优化")
rowan.set_project("CDK2 先导化合物优化")

# 后续工作流均归入此项目
wf = rowan.submit_descriptors_workflow("CCO", name="测试化合物")

# 后续检索
project = rowan.retrieve_project("CDK2 先导化合物优化")
workflows = rowan.list_workflows(project=project, size=50)
```

### 文件夹管理

```python
# 创建层级文件夹结构
folder = rowan.create_folder(name="docking/batch_1/screening")

wf = rowan.submit_docking_workflow(
    # ... 对接参数 ...
    folder=folder,
    name="compound_001",
)

# 列出文件夹内工作流
results = rowan.list_workflows(folder=folder)
```

## 工作流决策树

### pKa 与 MacropKa 选择

**使用微观 pKa 当：**

- 需获取单电离基团的 pKa
- 关注酸碱转变与质子化热力学
- 分子含 1-2 个电离位点
- 速度优先（更快，更省积分）

**使用 macropKa 当：**

- 需生理相关 pH 范围（如 0-14）内的 pH 依赖性行为
- 需跨 pH 的聚集电荷与质子化态分布
- 分子含多个耦合质子化的电离基团
- 需下游性质（如不同 pH 下水溶解度）

**决策示例：**

```text
苯酚 (pKa ~10)：用微观 pKa
胺类 (pKa ~9–10)：用微观 pKa
多电离位点药物（N、O、酸性基团）：用 macropKa
跨胃肠道 pH 的 ADME 评估：用 macropKa
```

### 构象搜索与互变异构搜索选择

**使用构象搜索当：**

- 已知单一互变异构形式
- 需多样化 3D 系综用于对接、MD 或 SAR 分析
- 化学空间以可旋转键为主

**使用互变异构搜索当：**

- 互变异构平衡不确定（如杂环、酮-烯醇体系）
- 需建模所有相关质子化异构体
- 下游计算（对接、pKa）依赖互变异构形式

**组合工作流：**

```python
# 步骤 1：寻找最佳互变异构体
taut_wf = rowan.submit_tautomer_search_workflow(
    initial_molecule="O=c1[nH]ccnc1",
    name="咪唑互变异构体",
)
best_taut = taut_wf.result().best_tautomer

# 步骤 2：基于最佳互变异构体生成构象
conf_wf = rowan.submit_conformer_search_workflow(
    initial_molecule=best_taut,
    name="咪唑构象",
)
```

### 对接 vs 类似物对接 vs 共折叠

| 工作流 | 适用场景 | 输入 | 输出 |
|----------|----------|-------|--------|
| 对接 | 单配体，已知结合口袋 | 蛋白质 + SMILES + 口袋坐标 | 构象、评分、dG |
| 类似物对接 | 5–100+ 相关化合物 | 蛋白质 + SMILES 列表 + 参考配体 | 所有构象，参考配体对齐 |
| 蛋白质-配体共折叠 | 序列 + 配体，无晶体结构 | 蛋白质序列 + SMILES | ML 预测结合复合物 |

## 常用工作流类别

### 1. 描述符计算

用于批量初筛、SAR 分析或探索性脚本的轻量级入口。

```python
wf = rowan.submit_descriptors_workflow(
    "CC(=O)Oc1ccccc1C(=O)O",  # 位置参数，接受 SMILES 字符串
    name="阿司匹林描述符",
)

result = wf.result()
print(result.descriptors['MW'])    # 180.16
print(result.descriptors['SLogP']) # 1.19
print(result.descriptors['TPSA'])  # 59.44
print(result.data['descriptors'])
# {'MW': 180.16, 'SLogP': 1.19, 'TPSA': 59.44, 'nHBDon': 1.0, 'nHBAcc': 4.0, ...}
```

**常用描述符键：**

| 键 | 描述 | 典型药物范围 |
|-----|-------------|-------------------|
| `MW` | 分子量 (Da) | <500 (Lipinski) |
| `SLogP` | 计算 LogP（亲脂性） | -2 至 +5 |
| `TPSA` | 拓扑极性表面积 (Å²) | <140（口服生物利用度） |
| `nHBDon` | 氢键供体数 | ≤5 (Lipinski) |
| `nHBAcc` | 氢键受体数 | ≤10 (Lipinski) |
| `nRot` | 可旋转键数 | <10（口服药物） |
| `nRing` | 环数 | — |
| `nHeavyAtom` | 重原子数 | — |
| `FilterItLogS` | 预估水溶解度 (LogS) | >-4 为佳 |
| `Lipinski` | Lipinski 五规则通过 (1.0) 或失败 (0.0) | — |

结果包含数百种额外分子描述符（BCUT、GETAWAY、WHIM 等），均可通过 `result.descriptors['key']` 访问。

### 2. 微观 pKa

用于特定结构的质子化态能量学与酸碱行为。

提供两种方法：

| 方法 | 输入 | 速度 | 覆盖范围 | 适用场景 |
|--------|-------|-------|--------|----------|
| `chemprop_nevolianis2025` | SMILES 字符串 | 快 | 仅去质子化（阴离子共轭碱） | 仅酸性基团；快速筛选 |
| `starling` | SMILES

```markdown
initial_smiles="CN1CCN(CC1)C2=NC=NC3=CC=CC=C32",  # 咪唑
    min_pH=0,
    max_pH=14,
    min_charge=-2,  # 默认值
    max_charge=2,   # 默认值
    compute_aqueous_solubility=True,  # 默认值
    name="imidazole macropKa",
)

result = wf.result()
print(result.pka_values)               # pKa值列表
print(result.logd_by_ph)               # {pH: logD}字典
print(result.aqueous_solubility_by_ph) # {pH: 溶解度}字典
print(result.isoelectric_point)        # 等电点
print(result.data)
# {'pKa_values': [...], 'logD_by_pH': {...}, 'aqueous_solubility_by_pH': {...}, ...}
```

### 4. 构象搜索

当需要高质量构象集合时进行3D构象生成。

```python
wf = rowan.submit_conformer_search_workflow(
    initial_molecule="CCOC(=O)N1CCC(CC1)Oc1ncnc2ccccc12",
    num_conformers=50,  # 可选：覆盖默认值
    name="构象搜索",
)

result = wf.result()
print(result.conformer_energies)  # [0.0, 1.2, 2.5, ...]
print(result.conformer_molecules)  # 3D分子列表
print(result.best_conformer)  # 最低能量构象
```

### 5. 互变异构体搜索

适用于杂环体系和互变异构状态影响下游建模的场景。

```python
wf = rowan.submit_tautomer_search_workflow(
    initial_molecule="O=c1[nH]ccnc1",  # 或酮式互变异构体
    name="咪唑酮互变异构体",
)

result = wf.result()
print(result.best_tautomer)  # 最稳定SMILES字符串
print(result.tautomers)      # 互变异构体SMILES列表
print(result.molecules)      # 分子对象列表
```

### 6. 分子对接

用于蛋白质-配体对接，可选姿势优化和构象生成。

```python
# 上传蛋白质一次，可在多个工作流中复用
protein = rowan.upload_protein(
    name="CDK2",
    file_path="cdk2.pdb",
)

# 定义结合口袋
pocket = {
    "center": [10.5, 24.2, 31.8],
    "size": [18.0, 18.0, 18.0],
}

# 提交对接任务
wf = rowan.submit_docking_workflow(
    protein=protein,
    pocket=pocket,
    initial_molecule="CCNc1ncc(c(Nc2ccc(F)cc2)n1)-c1cccnc1",
    do_pose_refinement=True,
    do_conformer_search=True,
    name="先导化合物对接",
)

result = wf.result()
print(result.scores)  # 对接分数(kcal/mol)
print(result.best_pose)  # 含3D坐标的分子对象
print(result.data)  # 原始结果字典
```

**蛋白质准备提示：**

- PDB文件应保持清洁（除非需要，否则移除水分子/杂原子）
- 在对接系列中使用相同蛋白质对象以保证一致性
- 若有PDB ID，改用`rowan.create_protein_from_pdb_id()`

### 7. 类似物对接

用于将化合物系列置于共享结合环境中。

```python
# 类似物系列（例如SAR研究）
analogues = [
    "CCNc1ncc(c(Nc2ccc(F)cc2)n1)-c1cccnc1",    # 参考化合物
    "CCNc1ncc(c(Nc2ccc(Cl)cc2)n1)-c1cccnc1",   # 氯代物
    "CCNc1ncc(c(Nc2ccc(OC)cc2)n1)-c1cccnc1",   # 甲氧基物
    "CCNc1ncc(c(Nc2cc(C)c(F)cc2)n1)-c1cccnc1", # 甲基氟代物
]

wf = rowan.submit_analogue_docking_workflow(
    analogues=analogues,
    initial_molecule=analogues[0],  # 参考配体
    protein=protein,
    pocket=pocket,
    name="SAR系列对接",
)

result = wf.result()
print(result.analogue_scores)  # 各类似物分数列表
print(result.best_poses)  # 最佳姿势列表
```

### 8. 多序列比对生成

用于多序列比对（对下游共折叠建模有用）。

```python
wf = rowan.submit_msa_workflow(
    initial_protein_sequences=[
        "MENFQKVEKIGEGTYGVVYKARNKLTGEVVALKKIRLDTETEGVP"
    ],
    output_formats=["colabfold", "chai", "boltz"],
    name="靶标MSA",
)

result = wf.result()
result.download_files()  # 下载比对文件到本地
```

### 9. 蛋白质-配体共折叠

当无晶体结构时用于AI驱动的复合物结合预测。

```python
wf = rowan.submit_protein_cofolding_workflow(
    initial_protein_sequences=[
        "MENFQKVEKIGEGTYGVVYKARNKLTGEVVALKKIRLDTETEGVP"
    ],
    initial_smiles_list=[
        "CCNc1ncc(c(Nc2ccc(F)cc2)n1)-c1cccnc1"
    ],
    name="蛋白质-配体共折叠",
)

result = wf.result()
print(result.predictions)  # 预测结构列表
print(result.messages)  # 模型元数据/警告

predicted_structure = result.get_predicted_structure()
predicted_structure.write("predicted_complex.pdb")
```

## 所有支持的工作流类型

所有工作流均遵循提交→等待→获取的模式，支持webhook和项目/文件夹管理。

### 核心分子建模工作流

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| 描述符 | `submit_descriptors_workflow` | 初筛：分子量、LogP、TPSA、HBA/HBD、Lipinski规则 |
| pKa | `submit_pka_workflow` | 单电离基团；需质子化热力学数据 |
| MacropKa | `submit_macropka_workflow` | 多电离药物；pH依赖性电荷/LogD/溶解度 |
| 构象搜索 | `submit_conformer_search_workflow` | 用于对接、MD或SAR的3D构象集合；已知互变异构体 |
| 互变异构体搜索 | `submit_tautomer_search_workflow` | 杂环、酮-烯醇体系；互变异构形式不确定 |
| 溶解度 | `submit_solubility_workflow` | 水或特定溶剂溶解度预测 |
| 膜渗透性 | `submit_membrane_permeability_workflow` | Caco-2、PAMPA、BBB、血浆渗透性 |
| ADMET | `submit_admet_workflow` | 全面药物相似性和ADMET性质扫描 |

### 基于结构的设计工作流

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| 对接 | `submit_docking_workflow` | 单配体，已知结合口袋 |
| 类似物对接 | `submit_analogue_docking_workflow` | SAR系列（5-100+化合物）在共享口袋中 |
| 批量对接 | `submit_batch_docking_workflow` | 快速库筛选；大型化合物集 |
| 蛋白质MD | `submit_protein_md_workflow` | 长时程动力学；构象采样 |
| 姿势分析MD | `submit_pose_analysis_md_workflow` | 对接姿势的MD优化 |
| 蛋白质共折叠 | `submit_protein_cofolding_workflow` | 无晶体结构；AI预测结合复合物 |
| 蛋白质结合剂设计 | `submit_protein_binder_design_workflow` | 针对蛋白质靶标的从头设计结合剂 |

### 高级计算化学

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| 基础计算 | `submit_basic_calculation_workflow` | QM/ML几何优化或单点能量计算 |
| 电子性质 | `submit_electronic_properties_workflow` | 偶极矩、部分电荷、HOMO-LUMO、ESP |
| BDE | `submit_bde_workflow` | 键解离能；代谢软点预测 |
| 氧化还原电位 | `submit_redox_potential_workflow` | 氧化/还原电位 |
| 自旋态 | `submit_spin_states_workflow` | 有机金属/自由基的自旋态能量排序 |
| 应变 | `submit_strain_workflow` | 相对于全局最小值的构象应变 |
| 扫描 | `submit_scan_workflow` | 势能面扫描；扭转剖面 |
| 多级优化 | `submit_multistage_opt_workflow` | 跨理论级别的渐进优化 |

### 反应化学

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| 双端过渡态搜索 | `submit_double_ended_ts_search_workflow` | 两个已知结构间的过渡态 |
| IRC | `submit_irc_workflow` | 确认过渡态连通性；内禀反应坐标 |

### 高级性质

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| NMR | `submit_nmr_workflow` | 预测1H/13C化学位移用于结构验证 |
| 离子淌度 | `submit_ion_mobility_workflow` | 碰撞截面(CCS)用于质谱方法开发 |
| 氢键强度 | `submit_hydrogen_bond_basicity_workflow` | 氢键供体/受体强度（用于制剂/溶解度） |
| 福井函数 | `submit_fukui_workflow` | 亲电/亲核攻击位点反应性指数 |
| 相互作用能分解 | `submit_interaction_energy_decomposition_workflow` | 片段级相互作用分析 |

### 结合自由能

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| RBFE/FEP | `submit_relative_binding_free_energy_perturbation_workflow` | 同源系列的相对ΔΔG计算 |
| RBFE图 | `submit_rbfe_graph_workflow` | 构建优化RBFE扰动网络 |

### 序列与结构生物学

| 工作流 | 函数 | 适用场景 |
|----------|----------|-------------|
| MSA | `submit_msa_workflow` | 用于共折叠的多序列比对（ColabFold, Chai, Boltz） |
| 溶剂依赖性构象 | `submit_solvent_dependent_conformers_workflow` | 溶剂化感知的构象集合 |

## 批量提交与获取

对于化合物库或类似物系列，使用特定工作流函数在循环中提交。通用函数`rowan.batch_submit_workflow()`和`rowan.submit_workflow()`当前会返回API的422错误——请改用命名函数（`submit_descriptors_workflow`、`submit_pka_workflow`等）。

### 批量提交

```python
smileses = ["CCO", "CC(=O)O", "c1ccccc1O"]
names = ["乙醇", "乙酸", "苯酚"]

workflows = [
    rowan.submit_descriptors_workflow(smi, name=name)
    for smi, name in zip(smileses, names)
]

print(f"已提交{len(workflows)}个工作流")
```

### 轮询批量状态

```python
statuses = rowan.batch_poll_status([wf.uuid for wf in workflows])
# 返回聚合计数（非按UUID）：
# {'queued': 0, 'running': 1, 'complete': 2, 'failed': 0, 'total': 3, ...}

if statuses["complete"] == statuses["total"]:
    print("所有工作流已完成")
elif statuses["failed"] > 0:
    print(f"{statuses['failed']}个工作流失败")
```

### 获取并收集结果

```python
results = []
for wf in workflows:
    try:
        result = wf.result()
        results.append(result.data)
    except rowan.WorkflowError as e:
        print(f"工作流{wf.uuid}失败: {e}")

# 可选：聚合到DataFrame
import pandas as pd
df = pd.DataFrame(results)
```

### 非阻塞/提交后检查模式

对于长时间运行的工作流，若不想保持进程开启，可提交工作流后保存UUID，在独立进程中稍后检查。

**会话1——提交并保存UUID：**

```python
import rowan, json

rowan.api_key = "..."
smileses = ["CCO", "CC(=O)O", "c1ccccc1O"]

workflows = [
    rowan.submit_descriptors_workflow(smi, name=f"化合物_{i}")
    for i, smi in enumerate(smileses)
]

# 保存UUID到磁盘（或数据库）
uuids = [wf.uuid for wf in workflows]
with open("workflow_uuids.json", "w") as f:
    json.dump(uuids, f)

print("已提交。请稍后检查。")
```

**会话2——准备就绪时检查状态并收集结果：**

```python
import rowan, json

rowan.api_key = "..."

with open("workflow_uuids.json") as f:
    uuids = json.load(f)

results = []
for uuid in uuids:
    wf = rowan.retrieve_workflow(uuid)
    if wf.done():
        result = wf.result(wait=False)
        results.append({"uuid": uuid, "data": result.data})
    else:
        print(f"{uuid}: 仍在运行({wf.status})")

print(f"已收集{len(results)}个完成结果")
```

## Webhook与异步工作流

对于长时间运行的任务或不想保持进程存活的场景，使用webhook在工作流完成时通知后端。

### 设置webhook

每个工作流提交函数都接受`webhook_url`参数：

```python
wf = rowan.submit_docking_workflow(
    protein=protein,
    pocket=pocket,
    initial_molecule="CCO",
    webhook_url="https://myserver.com/rowan_callback",
    name="带webhook的对接",
)

print(f"工作流已提交。完成后将POST结果到webhook。")
```

Webhook URL可传递给任何特定工作流函数（`submit_docking_workflow()`、`submit_pka_workflow()`、`submit_descriptors_workflow()`等）。

### 带密钥的webhook认证

Rowan支持webhook签名验证以确保请求真实性。您需要：

1. **创建或获取webhook密钥：**

```python
import rowan

# 创建新webhook密钥
secret = rowan.create_webhook_secret()
print(f"您的webhook密钥: {secret.secret}")

# 或获取现有密钥
secret = rowan.get_webhook_secret()

# 轮换密钥（使旧密钥失效，创建新密钥）
new_secret = rowan.rotate_webhook_secret()
```

2. **验证传入的webhook请求：**

```python
import rowan
import hmac
import json

def verify_webhook(request_body: bytes, signature: str, secret: str) -> bool:
    """验证webhook请求的HMAC-SHA256签名"""
    return rowan.verify_webhook_secret(request_body, signature, secret)
```

### Webhook负载与签名

工作流完成时，Rowan向您的webhook URL发送带签名的POST请求，头部包含：

```text
X-Rowan-Signature: <HMAC-SHA256签名>
```

请求体包含完整的工作流结果：

```json
{
  "workflow_uuid": "wf_12345abc",
  "workflow_type": "docking",
  "workflow_name": "lead docking",
  "status": "COMPLETED_OK",
  "created_at": "2025-04-01T12:00:00Z",
  "completed_at": "2025-04-01T12:15:30Z",
  "data": {
    "scores": [-8.2, -8.0

```python
raise HTTPException(status_code=401, detail="无效的webhook签名")

    # 解析并处理
    payload = json.loads(body)
    wf_uuid = payload["workflow_uuid"]
    status = payload["status"]

    if status == "COMPLETED_OK":
        print(f"工作流 {wf_uuid} 成功！")
        result_data = payload["data"]
        # 处理结果、更新数据库、触发下一个工作流等
    elif status == "FAILED":
        print(f"工作流 {wf_uuid} 失败！")
        # 处理失败情况

    # 快速响应以防止重试
    return {"status": "已接收"}
```

### Webhook最佳实践

- **始终验证签名**：使用`rowan.verify_webhook_secret()`确保请求来自Rowan
- **快速响应**（<5秒）：将繁重处理卸载到异步任务或后台作业
- **实现幂等性**：工作流可能重试；使用`workflow_uuid`优雅处理重复负载
- **记录所有事件**：用于调试和审计追踪
- **适用于长周期任务**：50+工作流时webhook优势明显；小型任务用`result()`轮询更简单
- **定期轮换密钥**：使用`rowan.rotate_webhook_secret()`增强安全性
- **返回2xx状态码**：确认接收；Rowan可能在5xx错误时重试

## 蛋白质工具

### 上传蛋白质

```python
# 从本地PDB文件上传
protein = rowan.upload_protein(
    name="egfr_kin酶结构域",
    file_path="egfr_kinase.pdb",
)

# 从PDB数据库创建
protein_from_pdb = rowan.create_protein_from_pdb_id(
    name="CDK2 (1M17)",
    code="1M17",
)

# 检索已上传蛋白质
protein = rowan.retrieve_protein("protein-uuid")

# 列出所有蛋白质
my_proteins = rowan.list_proteins()
```

### 蛋白质制备指南

- **文件格式**：PDB, mmCIF（Rowan自动识别）
- **水分子**：Rowan通常保留相关水分子；如需移除大量水请提前处理
- **杂原子**：辅因子、离子和结合配体通常保留；上传前移除不需要的杂原子
- **多链蛋白质**：完全支持
- **分辨率**：兼容NMR结构、同源模型和冷冻电镜；质量影响下游预测
- **验证**：Rowan验证PDB语法；严重格式错误文件可能被拒绝

## 端到端示例：先导化合物优化方案

此示例展示优化先导化合物的完整工作流：

```python
import rowan
import pandas as pd

# 1. 创建项目与文件夹进行组织
project = rowan.create_project(name="CDK2先导化合物优化")
rowan.set_project("CDK2先导化合物优化")
folder = rowan.create_folder(name="round_1_互变异构体与pKa")

# 2. 加载先导化合物及类似物
hit = "CCNc1ncc(c(Nc2ccc(F)cc2)n1)-c1cccnc1"  # 已知先导化合物
analogues = [
    "CCNc1ncc(c(Nc2ccccc2)n1)-c1cccnc1",      # 去除F
    "CCNc1ncc(c(Nc2ccc(Cl)cc2)n1)-c1cccnc1",  # Cl替代F
    "CCC(C)Nc1ncc(c(Nc2ccc(F)cc2)n1)-c1cccnc1",  # 丙基替代乙基
]

# 3. 确定最佳互变异构体（以防万一）
print("搜索互变异构体...")
taut_workflows = [
    rowan.submit_tautomer_search_workflow(
        smi, name=f"analog_{i}", folder=folder,
    )
    for i, smi in enumerate(analogues)
]

best_tautomers = []
for wf in taut_workflows:
    result = wf.result()
    best_tautomers.append(result.best_tautomer)

# 4. 预测所有类似物的pKa和基础性质
print("预测pKa与性质...")
pka_workflows = [
    rowan.submit_pka_workflow(
        smi, method="chemprop_nevolianis2025", name=f"pka_{i}", folder=folder,
    )
    for i, smi in enumerate(best_tautomers)
]

descriptor_workflows = [
    rowan.submit_descriptors_workflow(smi, name=f"desc_{i}", folder=folder)
    for i, smi in enumerate(best_tautomers)
]

# 5. 收集结果
pka_results = []
for wf in pka_workflows:
    try:
        result = wf.result()
        pka_results.append({
            "compound": wf.name,
            "pka": result.strongest_acid,  # 最强酸性位点的pKa
            "uuid": wf.uuid,
        })
    except rowan.WorkflowError as e:
        print(f"pKa预测失败 {wf.name}: {e}")

descriptor_results = []
for wf in descriptor_workflows:
    try:
        result = wf.result()
        desc = result.descriptors
        descriptor_results.append({
            "compound": wf.name,
            "mw": desc.get("MW"),
            "logp": desc.get("SLogP"),
            "hba": desc.get("nHBAcc"),
            "hbd": desc.get("nHBDon"),
            "uuid": wf.uuid,
        })
    except rowan.WorkflowError as e:
        print(f"描述符计算失败 {wf.name}: {e}")

# 6. 合并与总结
df_pka = pd.DataFrame(pka_results)
df_desc = pd.DataFrame(descriptor_results)
df = df_pka.merge(df_desc, on="compound", how="outer")

print("\n=== 初步构效关系 ===")
print(df.to_string())

# 7. 选择有潜力的化合物进行对接
# 化合物名称为"pka_0"、"pka_1"等——提取索引查找SMILES
top_idx = int(df.loc[df["pka"].idxmin(), "compound"].split("_")[1])
top_smiles = best_tautomers[top_idx]

print(f"\n进行对接: {top_smiles}")

# 8. 对接方案
protein = rowan.create_protein_from_pdb_id(name="CDK2_1CKP", code="1CKP")
pocket = {"center": [10.5, 24.2, 31.8], "size": [18.0, 18.0, 18.0]}

docking_wf = rowan.submit_docking_workflow(
    protein=protein,
    pocket=pocket,
    initial_molecule=top_smiles,
    do_pose_refinement=True,
    name=f"docking_{top_compound}",
)

dock_result = docking_wf.result()
print(f"\n对接分数: {dock_result.scores[0]:.2f} kcal/mol")
print(f"最佳构象保存至: best_pose.pdb")
dock_result.best_pose.write("best_pose.pdb")
```

## 错误处理与故障排除

### 常见错误及解决方案

```python
import rowan

# 错误1：无效SMILES
try:
    wf = rowan.submit_descriptors_workflow("CCCC(CC", name="无效smiles")  # 无效
except rowan.ValidationError as e:
    print(f"无效SMILES: {e}")
    # 解决方案：提交前用RDKit验证
    from rdkit import Chem
    smi = Chem.MolToSmiles(Chem.MolFromSmiles(smi))

# 错误2：未设置API密钥
try:
    wf = rowan.submit_descriptors_workflow("CCO")
except rowan.AuthenticationError:
    print("未找到API密钥。设置ROWAN_API_KEY环境变量或调用rowan.api_key='...'")

# 错误3：点数不足
try:
    wf = rowan.submit_protein_cofolding_workflow(...)
except rowan.InsufficientCreditsError as e:
    print(f"点数不足: {e}。请购买或减小任务规模")

# 错误4：工作流失败（分子问题等）
try:
    wf = rowan.submit_docking_workflow(...)
    result = wf.result()
except rowan.WorkflowError as e:
    print(f"工作流失败: {e}")
    # 查看wf.status获取详情
    print(f"状态: {wf.status}")

# 错误5：工作流未完成——手动轮询
result = wf.result(wait=True, poll_interval=5)  # 每5秒轮询等待
# 或非阻塞检查状态:
if not wf.done():
    print("工作流仍在运行。稍后调用wf.result()")
```

### 调试技巧

- **检查工作流状态**：`wf.status`、`wf.done()`或调用`wf.get_status()`
- **检查原始结果**：使用`result.data`而非便捷属性
- **重试失败工作流**：保存UUID并通过`rowan.retrieve_workflow(uuid)`重试
- **预先验证分子**：批量提交前使用RDKit或Chemaxon验证

## 推荐使用模式

- **优先使用Rowan原生工作流**而非底层组装
- **使用项目和文件夹**组织非简单任务（>5个工作流）
- **用`result()`阻塞至完成**（默认`wait=True, poll_interval=5`）
- **优先使用类型化结果属性**，未映射字段回退到`.data`
- **对化合物库或类似物系列使用批量提交**
- **链式工作流**用于多步骤化学任务：
  - `pKa → macropKa → 渗透性`（ADME评估）
  - `互变异构体搜索 → 对接 → 构象分析MD`（构象优化）
  - `多序列比对生成 → 蛋白-配体共折叠`（AI结构预测）
- **长周期任务使用webhook**（>50工作流）或异步流程
- **大尺度构象/对接搜索使用流式处理**获取交互反馈

## 总结

当工作流需要云执行分子设计任务时选用Rowan，尤其当您需要统一API和跨小分子建模、蛋白质、对接、ADME预测及ML结构生成的一致结果处理。

Rowan是分子设计工作流平台，不仅是远程化学引擎。它处理基础设施扩展、结果持久化和多步骤流程编排，让您专注于科研。
