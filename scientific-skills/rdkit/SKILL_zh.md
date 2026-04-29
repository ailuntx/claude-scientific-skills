---
name: rdkit
description: 用于精细分子控制的化学信息学工具包。支持SMILES/SDF解析、描述符计算（分子量、LogP、TPSA）、分子指纹、子结构搜索、2D/3D结构生成、相似性计算及化学反应处理。标准工作流建议使用datamol（RDKit封装接口），高级控制、自定义净化及专用算法请直接使用rdkit。
license: BSD-3-Clause license
metadata:
    skill-author: K-Dense Inc.
---

# RDKit 化学信息学工具包

## 概述

RDKit 是一个全面的化学信息学库，提供用于分子分析与操作的 Python API。本技能涵盖分子结构读写、描述符计算、指纹生成、子结构搜索、化学反应处理、2D/3D坐标生成及分子可视化等核心功能。适用于药物发现、计算化学和化学信息学研究任务。

## 核心功能

### 1. 分子输入输出与创建

**读取分子：**

从多种格式读取分子结构：

```python
from rdkit import Chem

# 从SMILES字符串读取
mol = Chem.MolFromSmiles('Cc1ccccc1')  # 返回Mol对象或None

# 从MOL文件读取
mol = Chem.MolFromMolFile('path/to/file.mol')

# 从MOL块（字符串数据）读取
mol = Chem.MolFromMolBlock(mol_block_string)

# 从InChI读取
mol = Chem.MolFromInchi('InChI=1S/C6H6/c1-2-4-6-5-3-1/h1-6H')
```

**输出分子：**

将分子转换为文本表示：

```python
# 转换为规范SMILES
smiles = Chem.MolToSmiles(mol)

# 转换为MOL块
mol_block = Chem.MolToMolBlock(mol)

# 转换为InChI
inchi = Chem.MolToInchi(mol)
```

**批量处理：**

处理多分子时使用Supplier/Writer对象：

```python
# 读取SDF文件
suppl = Chem.SDMolSupplier('molecules.sdf')
for mol in suppl:
    if mol is not None:  # 检查解析错误
        # 处理分子
        pass

# 读取SMILES文件
suppl = Chem.SmilesMolSupplier('molecules.smi', titleLine=False)

# 处理大文件或压缩数据
with gzip.open('molecules.sdf.gz') as f:
    suppl = Chem.ForwardSDMolSupplier(f)
    for mol in suppl:
        # 处理分子
        pass

# 大数据集多线程处理
suppl = Chem.MultithreadedSDMolSupplier('molecules.sdf')

# 写入SDF文件
writer = Chem.SDWriter('output.sdf')
for mol in molecules:
    writer.write(mol)
writer.close()
```

**重要提示：**
- 所有`MolFrom*`函数失败时返回`None`并附带错误信息
- 处理前务必检查`None`值
- 导入时自动执行分子净化（验证化合价、识别芳香性）

### 2. 分子净化与验证

RDKit在解析时自动执行13步净化流程，包括化合价检查、芳香性识别和手性分配。

**净化控制：**

```python
# 禁用自动净化
mol = Chem.MolFromSmiles('C1=CC=CC=C1', sanitize=False)

# 手动净化
Chem.SanitizeMol(mol)

# 净化前检测问题
problems = Chem.DetectChemistryProblems(mol)
for problem in problems:
    print(problem.GetType(), problem.Message())

# 部分净化（跳过特定步骤）
from rdkit.Chem import rdMolStandardize
Chem.SanitizeMol(mol, sanitizeOps=Chem.SANITIZE_ALL ^ Chem.SANITIZE_PROPERTIES)
```

**常见净化问题：**
- 原子显式化合价超限将引发异常
- 无效芳香环导致凯库勒化错误
- 未明确指定时自由基电子可能分配异常

### 3. 分子分析与属性

**访问分子结构：**

```python
# 遍历原子与化学键
for atom in mol.GetAtoms():
    print(atom.GetSymbol(), atom.GetIdx(), atom.GetDegree())

for bond in mol.GetBonds():
    print(bond.GetBeginAtomIdx(), bond.GetEndAtomIdx(), bond.GetBondType())

# 环信息
ring_info = mol.GetRingInfo()
ring_info.NumRings()
ring_info.AtomRings()  # 返回原子索引元组

# 检查原子是否在环内
atom = mol.GetAtomWithIdx(0)
atom.IsInRing()
atom.IsInRingSize(6)  # 检查六元环

# 寻找最小环集合（SSSR）
from rdkit.Chem import GetSymmSSSR
rings = GetSymmSSSR(mol)
```

**立体化学：**

```python
# 寻找手性中心
from rdkit.Chem import FindMolChiralCenters
chiral_centers = FindMolChiralCenters(mol, includeUnassigned=True)
# 返回(原子索引, 手性)元组列表

# 从3D坐标分配立体化学
from rdkit.Chem import AssignStereochemistryFrom3D
AssignStereochemistryFrom3D(mol)

# 检查键立体化学
bond = mol.GetBondWithIdx(0)
stereo = bond.GetStereo()  # STEREONONE, STEREOZ, STEREOE等
```

**片段分析：**

```python
# 获取不连续片段
frags = Chem.GetMolFrags(mol, asMols=True)

# 在特定键上断裂
from rdkit.Chem import FragmentOnBonds
frag_mol = FragmentOnBonds(mol, [bond_idx1, bond_idx2])

# 统计环系
from rdkit.Chem.Scaffolds import MurckoScaffold
scaffold = MurckoScaffold.GetScaffoldForMol(mol)
```

### 4. 分子描述符与物性

**基础描述符：**

```python
from rdkit.Chem import Descriptors

# 分子量
mw = Descriptors.MolWt(mol)
exact_mw = Descriptors.ExactMolWt(mol)

# LogP（亲脂性）
logp = Descriptors.MolLogP(mol)

# 拓扑极性表面积
tpsa = Descriptors.TPSA(mol)

# 氢键供体/受体数量
hbd = Descriptors.NumHDonors(mol)
hba = Descriptors.NumHAcceptors(mol)

# 可旋转键数量
rot_bonds = Descriptors.NumRotatableBonds(mol)

# 芳香环数量
aromatic_rings = Descriptors.NumAromaticRings(mol)
```

**批量描述符计算：**

```python
# 一次性计算所有描述符
all_descriptors = Descriptors.CalcMolDescriptors(mol)
# 返回字典: {'MolWt': 180.16, 'MolLogP': 1.23, ...}

# 获取可用描述符名称列表
descriptor_names = [desc[0] for desc in Descriptors._descList]
```

**类药五原则：**

```python
# 检查类药性
mw = Descriptors.MolWt(mol) <= 500
logp = Descriptors.MolLogP(mol) <= 5
hbd = Descriptors.NumHDonors(mol) <= 5
hba = Descriptors.NumHAcceptors(mol) <= 10

is_drug_like = mw and logp and hbd and hba
```

### 5. 分子指纹与相似性

**指纹类型：**

```python
from rdkit.Chem import rdFingerprintGenerator
from rdkit.Chem import MACCSkeys

# RDKit拓扑指纹
rdk_gen = rdFingerprintGenerator.GetRDKitFPGenerator(minPath=1, maxPath=7, fpSize=2048)
fp = rdk_gen.GetFingerprint(mol)

# Morgan指纹（圆形指纹，类似ECFP）
# 使用rdFingerprintGenerator的现代API
morgan_gen = rdFingerprintGenerator.GetMorganGenerator(radius=2, fpSize=2048)
fp = morgan_gen.GetFingerprint(mol)
# 计数指纹
fp_count = morgan_gen.GetCountFingerprint(mol)

# MACCS密钥（166位结构密钥）
fp = MACCSkeys.GenMACCSKeys(mol)

# 原子对指纹
ap_gen = rdFingerprintGenerator.GetAtomPairGenerator()
fp = ap_gen.GetFingerprint(mol)

# 拓扑扭转指纹
tt_gen = rdFingerprintGenerator.GetTopologicalTorsionGenerator()
fp = tt_gen.GetFingerprint(mol)

# Avalon指纹（若可用）
from rdkit.Avalon import pyAvalonTools
fp = pyAvalonTools.GetAvalonFP(mol)
```

**相似性计算：**

```python
from rdkit import DataStructs
from rdkit.Chem import rdFingerprintGenerator

# 使用生成器创建指纹
mfpgen = rdFingerprintGenerator.GetMorganGenerator(radius=2, fpSize=2048)
fp1 = mfpgen.GetFingerprint(mol1)
fp2 = mfpgen.GetFingerprint(mol2)

# 计算Tanimoto相似度
similarity = DataStructs.TanimotoSimilarity(fp1, fp2)

# 多分子相似性计算
fps = [mfpgen.GetFingerprint(m) for m in [mol2, mol3, mol4]]
similarities = DataStructs.BulkTanimotoSimilarity(fp1, fps)

# 其他相似性指标
dice = DataStructs.DiceSimilarity(fp1, fp2)
cosine = DataStructs.CosineSimilarity(fp1, fp2)
```

**聚类与多样性：**

```python
# 基于指纹相似度的Butina聚类
from rdkit.ML.Cluster import Butina

# 计算距离矩阵
dists = []
mfpgen = rdFingerprintGenerator.GetMorganGenerator(radius=2, fpSize=2048)
fps = [mfpgen.GetFingerprint(mol) for mol in mols]
for i in range(len(fps)):
    sims = DataStructs.BulkTanimotoSimilarity(fps[i], fps[:i])
    dists.extend([1-sim for sim in sims])

# 按距离阈值聚类
clusters = Butina.ClusterData(dists, len(fps), distThresh=0.3, isDistData=True)
```

### 6. 子结构搜索与SMARTS

**基础子结构匹配：**

```python
# 使用SMARTS定义查询
query = Chem.MolFromSmarts('[#6]1:[#6]:[#6]:[#6]:[#6]:[#6]:1')  # 苯环

# 检查分子是否包含子结构
has_match = mol.HasSubstructMatch(query)

# 获取所有匹配（返回原子索引元组）
matches = mol.GetSubstructMatches(query)

# 仅获取首个匹配
match = mol.GetSubstructMatch(query)
```

**常用SMARTS模式：**

```python
# 伯醇
primary_alcohol = Chem.MolFromSmarts('[CH2][OH1]')

# 羧酸
carboxylic_acid = Chem.MolFromSmarts('C(=O)[OH]')

# 酰胺
amide = Chem.MolFromSmarts('C(=O)N')

# 芳香杂环
aromatic_n = Chem.MolFromSmarts('[nR]')  # 环内芳香氮

# 大环（>12原子）
macrocycle = Chem.MolFromSmarts('[r{12-}]')
```

**匹配规则：**
- 查询中未指定属性可匹配目标任意值
- 除非显式指定，否则忽略氢原子
- 带电荷查询原子不匹配中性目标原子
- 芳香性查询原子不匹配脂肪族目标原子（通用查询除外）

### 7. 化学反应

**反应SMARTS：**

```python
from rdkit.Chem import AllChem

# 用SMARTS定义反应: 反应物 >> 产物
rxn = AllChem.ReactionFromSmarts('[C:1]=[O:2]>>[C:1][O:2]')  # 酮还原

# 对分子应用反应
reactants = (mol1,)
products = rxn.RunReactants(reactants)

# 产物为元组的元组（每组产物一个元组）
for product_set in products:
    for product in product_set:
        # 净化产物
        Chem.SanitizeMol(product)
```

**反应特性：**
- 原子映射保留反应物与产物间特定原子
- 产物中的虚拟原子由对应反应物原子替换
- "任意"键继承反应物键级
- 除非显式更改，否则保留手性

**反应相似性：**

```python
# 生成反应指纹
fp = AllChem.CreateDifferenceFingerprintForReaction(rxn)

# 比较反应
similarity = DataStructs.TanimotoSimilarity(fp1, fp2)
```

### 8. 2D与3D坐标生成

**2D坐标生成：**

```python
from rdkit.Chem import AllChem

# 为分子绘制生成2D坐标
AllChem.Compute2DCoords(mol)

# 对齐模板结构
template = Chem.MolFromSmiles('c1ccccc1')
AllChem.Compute2DCoords(template)
AllChem.GenerateDepictionMatching2DStructure(mol, template)
```

**3D坐标生成与构象：**

```python
# 使用ETKDG生成单构象
AllChem.EmbedMolecule(mol, randomSeed=42)

# 生成多构象
conf_ids = AllChem.EmbedMultipleConfs(mol, numConfs=10, randomSeed=42)

# 力场优化几何结构
AllChem.UFFOptimizeMolecule(mol)  # UFF力场
AllChem.MMFFOptimizeMolecule(mol)  # MMFF94力场

# 优化所有构象
for conf_id in conf_ids:
    AllChem.MMFFOptimizeMolecule(mol, confId=conf_id)

# 计算构象间RMSD
from rdkit.Chem import AllChem
rms = AllChem.GetConformerRMS(mol, conf_id1, conf_id2)

# 分子对齐
AllChem.AlignMol(probe_mol, ref_mol)
```

**约束嵌入：**

```python
# 将分子部分结构约束到特定坐标嵌入
AllChem.ConstrainedEmbed(mol, core_mol)
```

### 9. 分子可视化

**基础绘图：**

```python
from rdkit.Chem import Draw

# 绘制单分子到PIL图像
img = Draw.MolToImage(mol, size=(300, 300))
img.save('molecule.png')

# 直接输出到文件
Draw.MolToFile(mol, 'molecule.png')

# 网格布局绘制多分子
mols = [mol1, mol2, mol3, mol4]
img = Draw.MolsToGridImage(mols, molsPerRow=2, subImgSize=(200, 200))
```

**高亮子结构：**

```python
# 高亮子结构匹配
query = Chem.MolFromSmarts('c1ccccc1')
match = mol.GetSubstructMatch(query)

img = Draw.MolToImage(mol, highlightAtoms=match)

# 自定义高亮颜色
highlight_colors = {atom_idx: (1, 0, 0) for atom_idx in match}  # 红色
img = Draw.MolToImage(mol, highlightAtoms=match,
                      highlightAtomColors=highlight_colors)
```

**可视化定制：**

```python
from rdkit.Chem.Draw import rdMolDraw2D

# 创建带定制选项的绘图器
drawer = rdMolDraw2D.MolDraw2DCairo(300, 300)
opts = drawer.drawOptions()

# 定制选项
opts.addAtomIndices = True
opts.addStereoAnnotation = True
opts.bondLineWidth = 2

# 绘制分子
drawer.DrawMolecule(mol)
drawer.FinishDrawing()

# 保存文件
with open('molecule.png', 'wb') as f:
    f.write(drawer.GetDrawingText())
```

**Jupyter Notebook集成：**

```python
# 在Jupyter中启用内联显示
from rdkit.Chem.Draw import IPythonConsole

# 定制默认显示
IPythonConsole.ipython_useSVG = True  # 使用SVG替代PNG
IP

```markdown
mol = Chem.RemoveHs(mol_h)
```

**凯库勒化与芳香性处理：**

```python
# 将芳香键转换为交替的单/双键
Chem.Kekulize(mol)

# 设置芳香性
Chem.SetAromaticity(mol)
```

**子结构替换：**

```python
# 用新结构替换子结构
query = Chem.MolFromSmarts('c1ccccc1')  # 苯环
replacement = Chem.MolFromSmiles('C1CCCCC1')  # 环己烷

new_mol = Chem.ReplaceSubstructs(mol, query, replacement)[0]
```

**电荷中和：**

```python
# 通过添加/移除氢原子消除形式电荷
from rdkit.Chem.MolStandardize import rdMolStandardize

# 使用电荷中和器
uncharger = rdMolStandardize.Uncharger()
mol_neutral = uncharger.uncharge(mol)
```

### 11. 分子哈希与标准化处理

**分子哈希生成：**

```python
from rdkit.Chem import rdMolHash

# 生成Murcko骨架哈希
scaffold_hash = rdMolHash.MolHash(mol, rdMolHash.HashFunction.MurckoScaffold)

# 生成规范SMILES哈希
canonical_hash = rdMolHash.MolHash(mol, rdMolHash.HashFunction.CanonicalSmiles)

# 生成区域异构体哈希（忽略立体化学）
regio_hash = rdMolHash.MolHash(mol, rdMolHash.HashFunction.Regioisomer)
```

**随机化SMILES：**

```python
# 生成随机SMILES表示（用于数据增强）
from rdkit.Chem import MolToRandomSmilesVect

random_smiles = MolToRandomSmilesVect(mol, numSmiles=10, randomSeed=42)
```

### 12. 药效团与3D特征

**药效团特征提取：**

```python
from rdkit.Chem import ChemicalFeatures
from rdkit import RDConfig
import os

# 加载特征工厂
fdef_path = os.path.join(RDConfig.RDDataDir, 'BaseFeatures.fdef')
factory = ChemicalFeatures.BuildFeatureFactory(fdef_path)

# 获取药效团特征
features = factory.GetFeaturesForMol(mol)

for feat in features:
    print(feat.GetFamily(), feat.GetType(), feat.GetAtomIds())
```

## 常用工作流

### 类药性分析

```python
from rdkit import Chem
from rdkit.Chem import Descriptors

def analyze_druglikeness(smiles):
    mol = Chem.MolFromSmiles(smiles)
    if mol is None:
        return None

    # 计算Lipinski描述符
    results = {
        'MW': Descriptors.MolWt(mol),
        'LogP': Descriptors.MolLogP(mol),
        'HBD': Descriptors.NumHDonors(mol),
        'HBA': Descriptors.NumHAcceptors(mol),
        'TPSA': Descriptors.TPSA(mol),
        'RotBonds': Descriptors.NumRotatableBonds(mol)
    }

    # 检查Lipinski五规则
    results['Lipinski'] = (
        results['MW'] <= 500 and
        results['LogP'] <= 5 and
        results['HBD'] <= 5 and
        results['HBA'] <= 10
    )

    return results
```

### 相似性筛选

```python
from rdkit import Chem
from rdkit.Chem import AllChem
from rdkit import DataStructs

def similarity_screen(query_smiles, database_smiles, threshold=0.7):
    query_mol = Chem.MolFromSmiles(query_smiles)
    query_fp = AllChem.GetMorganFingerprintAsBitVect(query_mol, 2)

    hits = []
    for idx, smiles in enumerate(database_smiles):
        mol = Chem.MolFromSmiles(smiles)
        if mol:
            fp = AllChem.GetMorganFingerprintAsBitVect(mol, 2)
            sim = DataStructs.TanimotoSimilarity(query_fp, fp)
            if sim >= threshold:
                hits.append((idx, smiles, sim))

    return sorted(hits, key=lambda x: x[2], reverse=True)
```

### 子结构过滤

```python
from rdkit import Chem

def filter_by_substructure(smiles_list, pattern_smarts):
    query = Chem.MolFromSmarts(pattern_smarts)

    hits = []
    for smiles in smiles_list:
        mol = Chem.MolFromSmiles(smiles)
        if mol and mol.HasSubstructMatch(query):
            hits.append(smiles)

    return hits
```

## 最佳实践

### 错误处理

解析分子时始终检查`None`值：

```python
mol = Chem.MolFromSmiles(smiles)
if mol is None:
    print(f"解析失败: {smiles}")
    continue
```

### 性能优化

**使用二进制格式存储：**

```python
import pickle

# 使用pickle存储分子以实现快速加载
with open('molecules.pkl', 'wb') as f:
    pickle.dump(mols, f)

# 加载pickle分子（比重新解析快得多）
with open('molecules.pkl', 'rb') as f:
    mols = pickle.load(f)
```

**使用批量操作：**

```python
# 一次性计算所有分子的指纹
fps = [AllChem.GetMorganFingerprintAsBitVect(mol, 2) for mol in mols]

# 使用批量相似度计算
similarities = DataStructs.BulkTanimotoSimilarity(fps[0], fps[1:])
```

### 线程安全

RDKit操作在以下场景通常是线程安全的：
- 分子I/O（SMILES, mol文件块）
- 坐标生成
- 指纹和描述符计算
- 子结构搜索
- 化学反应
- 分子绘制

**非线程安全：** 并发访问时的MolSuppliers对象

### 内存管理

处理大型数据集时：

```python
# 使用ForwardSDMolSupplier避免加载整个文件
with open('large.sdf') as f:
    suppl = Chem.ForwardSDMolSupplier(f)
    for mol in suppl:
        # 逐分子处理
        pass

# 使用MultithreadedSDMolSupplier实现并行处理
suppl = Chem.MultithreadedSDMolSupplier('large.sdf', numWriterThreads=4)
```

## 常见陷阱

1. **未检查None值：** 解析后始终验证分子对象
2. **化学验证失败：** 使用`DetectChemistryProblems()`调试
3. **缺失氢原子：** 计算依赖氢的性质时使用`AddHs()`
4. **2D与3D混淆：** 可视化或3D分析前生成对应坐标
5. **SMARTS匹配规则：** 注意未指定属性会匹配任意值
6. **MolSuppliers线程安全：** 避免跨线程共享supplier对象

## 资源

### references/

本技能包含详细API参考文档：

- `api_reference.md` - 按功能组织的RDKit模块/函数/类完整列表
- `descriptors_reference.md` - 带说明的分子描述符全集
- `smarts_patterns.md` - 官能团和结构特征的常用SMARTS模式

需要具体API细节、参数信息或模式示例时加载这些参考文档

### scripts/

常用RDKit工作流示例脚本：

- `molecular_properties.py` - 计算综合分子属性和描述符
- `similarity_search.py` - 基于指纹的相似性筛选
- `substructure_filter.py` - 通过子结构模式过滤分子

这些脚本可直接执行或作为自定义工作流模板
```
