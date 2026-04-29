# RDKit API 参考

本文档按功能组织，提供 RDKit Python API 的全面参考。

## 核心模块：rdkit.Chem

处理分子的基础模块。

### 分子输入输出

**读取分子：**

- `Chem.MolFromSmiles(smiles, sanitize=True)` - 解析 SMILES 字符串
- `Chem.MolFromSmarts(smarts)` - 解析 SMARTS 模式
- `Chem.MolFromMolFile(filename, sanitize=True, removeHs=True)` - 读取 MOL 文件
- `Chem.MolFromMolBlock(molblock, sanitize=True, removeHs=True)` - 解析 MOL 块字符串
- `Chem.MolFromMol2File(filename, sanitize=True, removeHs=True)` - 读取 MOL2 文件
- `Chem.MolFromMol2Block(molblock, sanitize=True, removeHs=True)` - 解析 MOL2 块
- `Chem.MolFromPDBFile(filename, sanitize=True, removeHs=True)` - 读取 PDB 文件
- `Chem.MolFromPDBBlock(pdbblock, sanitize=True, removeHs=True)` - 解析 PDB 块
- `Chem.MolFromInchi(inchi, sanitize=True, removeHs=True)` - 解析 InChI 字符串
- `Chem.MolFromSequence(seq, sanitize=True)` - 从肽序列创建分子

**写入分子：**

- `Chem.MolToSmiles(mol, isomericSmiles=True, canonical=True)` - 转换为 SMILES
- `Chem.MolToSmarts(mol, isomericSmarts=False)` - 转换为 SMARTS
- `Chem.MolToMolBlock(mol, includeStereo=True, confId=-1)` - 转换为 MOL 块
- `Chem.MolToMolFile(mol, filename, includeStereo=True, confId=-1)` - 写入 MOL 文件
- `Chem.MolToPDBBlock(mol, confId=-1)` - 转换为 PDB 块
- `Chem.MolToPDBFile(mol, filename, confId=-1)` - 写入 PDB 文件
- `Chem.MolToInchi(mol, options='')` - 转换为 InChI
- `Chem.MolToInchiKey(mol, options='')` - 生成 InChI 密钥
- `Chem.MolToSequence(mol)` - 转换为肽序列

**批量输入输出：**

- `Chem.SDMolSupplier(filename, sanitize=True, removeHs=True)` - SDF 文件读取器
- `Chem.ForwardSDMolSupplier(fileobj, sanitize=True, removeHs=True)` - 仅向前 SDF 读取器
- `Chem.MultithreadedSDMolSupplier(filename, numWriterThreads=1)` - 并行 SDF 读取器
- `Chem.SmilesMolSupplier(filename, delimiter=' ', titleLine=True)` - SMILES 文件读取器
- `Chem.SDWriter(filename)` - SDF 文件写入器
- `Chem.SmilesWriter(filename, delimiter=' ', includeHeader=True)` - SMILES 文件写入器

### 分子操作

**净化处理：**

- `Chem.SanitizeMol(mol, sanitizeOps=SANITIZE_ALL, catchErrors=False)` - 净化分子
- `Chem.DetectChemistryProblems(mol, sanitizeOps=SANITIZE_ALL)` - 检测净化问题
- `Chem.AssignStereochemistry(mol, cleanIt=True, force=False)` - 分配立体化学
- `Chem.FindPotentialStereo(mol)` - 查找潜在立体中心
- `Chem.AssignStereochemistryFrom3D(mol, confId=-1)` - 从 3D 坐标分配立体化学

**氢原子管理：**

- `Chem.AddHs(mol, explicitOnly=False, addCoords=False)` - 添加显式氢原子
- `Chem.RemoveHs(mol, implicitOnly=False, updateExplicitCount=False)` - 移除氢原子
- `Chem.RemoveAllHs(mol)` - 移除所有氢原子

**芳香性：**

- `Chem.SetAromaticity(mol, model=AROMATICITY_RDKIT)` - 设置芳香性模型
- `Chem.Kekulize(mol, clearAromaticFlags=False)` - 凯库勒化芳香键
- `Chem.SetConjugation(mol)` - 设置共轭标志

**片段处理：**

- `Chem.GetMolFrags(mol, asMols=False, sanitizeFrags=True)` - 获取不连续片段
- `Chem.FragmentOnBonds(mol, bondIndices, addDummies=True)` - 在特定键上断裂
- `Chem.ReplaceSubstructs(mol, query, replacement, replaceAll=False)` - 替换子结构
- `Chem.DeleteSubstructs(mol, query, onlyFrags=False)` - 删除子结构

**立体化学：**

- `Chem.FindMolChiralCenters(mol, includeUnassigned=False, useLegacyImplementation=False)` - 查找手性中心
- `Chem.FindPotentialStereo(mol, cleanIt=True)` - 查找潜在立体中心

### 子结构搜索

**基础匹配：**

- `mol.HasSubstructMatch(query, useChirality=False)` - 检查子结构匹配
- `mol.GetSubstructMatch(query, useChirality=False)` - 获取首个匹配
- `mol.GetSubstructMatches(query, uniquify=True, useChirality=False)` - 获取所有匹配
- `mol.GetSubstructMatches(query, maxMatches=1000)` - 限制匹配数量

### 分子属性

**原子方法：**

- `atom.GetSymbol()` - 原子符号
- `atom.GetAtomicNum()` - 原子序数
- `atom.GetDegree()` - 键的数量
- `atom.GetTotalDegree()` - 包含氢原子
- `atom.GetFormalCharge()` - 形式电荷
- `atom.GetNumRadicalElectrons()` - 自由基电子数
- `atom.GetIsAromatic()` - 芳香性标志
- `atom.GetHybridization()` - 杂化类型 (SP, SP2, SP3 等)
- `atom.GetIdx()` - 原子索引
- `atom.IsInRing()` - 是否在环中
- `atom.IsInRingSize(size)` - 是否在特定大小环中
- `atom.GetChiralTag()` - 手性标签

**键方法：**

- `bond.GetBondType()` - 键类型 (单键, 双键, 三键, 芳香键)
- `bond.GetBeginAtomIdx()` - 起始原子索引
- `bond.GetEndAtomIdx()` - 终止原子索引
- `bond.GetIsConjugated()` - 共轭标志
- `bond.GetIsAromatic()` - 芳香性标志
- `bond.IsInRing()` - 是否在环中
- `bond.GetStereo()` - 立体化学 (无立体, Z型, E型 等)

**分子方法：**

- `mol.GetNumAtoms(onlyExplicit=True)` - 原子数量
- `mol.GetNumHeavyAtoms()` - 重原子数量
- `mol.GetNumBonds()` - 键的数量
- `mol.GetAtoms()` - 原子迭代器
- `mol.GetBonds()` - 键迭代器
- `mol.GetAtomWithIdx(idx)` - 获取特定原子
- `mol.GetBondWithIdx(idx)` - 获取特定键
- `mol.GetRingInfo()` - 环信息对象

**环信息：**

- `Chem.GetSymmSSSR(mol)` - 获取最小环最小集
- `Chem.GetSSSR(mol)` - GetSymmSSSR 的别名
- `ring_info.NumRings()` - 环的数量
- `ring_info.AtomRings()` - 环中原子索引元组
- `ring_info.BondRings()` - 环中键索引元组

## rdkit.Chem.AllChem

扩展化学功能。

### 2D/3D 坐标生成

- `AllChem.Compute2DCoords(mol, canonOrient=True, clearConfs=True)` - 生成 2D 坐标
- `AllChem.EmbedMolecule(mol, maxAttempts=0, randomSeed=-1, useRandomCoords=False)` - 生成 3D 构象
- `AllChem.EmbedMultipleConfs(mol, numConfs=10, maxAttempts=0, randomSeed=-1)` - 生成多个构象
- `AllChem.ConstrainedEmbed(mol, core, useTethers=True)` - 约束嵌入
- `AllChem.GenerateDepictionMatching2DStructure(mol, reference, refPattern=None)` - 对齐模板

### 力场优化

- `AllChem.UFFOptimizeMolecule(mol, maxIters=200, confId=-1)` - UFF 优化
- `AllChem.MMFFOptimizeMolecule(mol, maxIters=200, confId=-1, mmffVariant='MMFF94')` - MMFF 优化
- `AllChem.UFFGetMoleculeForceField(mol, confId=-1)` - 获取 UFF 力场对象
- `AllChem.MMFFGetMoleculeForceField(mol, pyMMFFMolProperties, confId=-1)` - 获取 MMFF 力场

### 构象分析

- `AllChem.GetConformerRMS(mol, confId1, confId2, prealigned=False)` - 计算 RMSD
- `AllChem.GetConformerRMSMatrix(mol, prealigned=False)` - RMSD 矩阵
- `AllChem.AlignMol(prbMol, refMol, prbCid=-1, refCid=-1)` - 对齐分子
- `AllChem.AlignMolConformers(mol)` - 对齐所有构象

### 化学反应

- `AllChem.ReactionFromSmarts(smarts, useSmiles=False)` - 从 SMARTS 创建反应
- `reaction.RunReactants(reactants)` - 应用反应
- `reaction.RunReactant(reactant, reactionIdx)` - 应用于特定反应物
- `AllChem.CreateDifferenceFingerprintForReaction(reaction)` - 反应指纹

### 分子指纹

- `AllChem.GetMorganFingerprint(mol, radius, useFeatures=False)` - Morgan 指纹
- `AllChem.GetMorganFingerprintAsBitVect(mol, radius, nBits=2048)` - Morgan 位向量
- `AllChem.GetHashedMorganFingerprint(mol, radius, nBits=2048)` - 哈希 Morgan 指纹
- `AllChem.GetErGFingerprint(mol)` - ErG 指纹

## rdkit.Chem.Descriptors

分子描述符计算。

### 常用描述符

- `Descriptors.MolWt(mol)` - 分子量
- `Descriptors.ExactMolWt(mol)` - 精确分子量
- `Descriptors.HeavyAtomMolWt(mol)` - 重原子分子量
- `Descriptors.MolLogP(mol)` - LogP (亲脂性)
- `Descriptors.MolMR(mol)` - 摩尔折射率
- `Descriptors.TPSA(mol)` - 拓扑极性表面积
- `Descriptors.NumHDonors(mol)` - 氢键供体数
- `Descriptors.NumHAcceptors(mol)` - 氢键受体数
- `Descriptors.NumRotatableBonds(mol)` - 可旋转键数
- `Descriptors.NumAromaticRings(mol)` - 芳香环数
- `Descriptors.NumSaturatedRings(mol)` - 饱和环数
- `Descriptors.NumAliphaticRings(mol)` - 脂肪环数
- `Descriptors.NumAromaticHeterocycles(mol)` - 芳香杂环数
- `Descriptors.NumRadicalElectrons(mol)` - 自由基电子数
- `Descriptors.NumValenceElectrons(mol)` - 价电子数

### 批量计算

- `Descriptors.CalcMolDescriptors(mol)` - 计算所有描述符为字典

### 描述符列表

- `Descriptors._descList` - 所有描述符的 (名称, 函数) 元组列表

## rdkit.Chem.Draw

分子可视化。

### 图像生成

- `Draw.MolToImage(mol, size=(300,300), kekulize=True, wedgeBonds=True, highlightAtoms=None)` - 生成 PIL 图像
- `Draw.MolToFile(mol, filename, size=(300,300), kekulize=True, wedgeBonds=True)` - 保存到文件
- `Draw.MolsToGridImage(mols, molsPerRow=3, subImgSize=(200,200), legends=None)` - 分子网格
- `Draw.MolsMatrixToGridImage(mols, molsPerRow=3, subImgSize=(200,200), legends=None)` - 嵌套网格
- `Draw.ReactionToImage(rxn, subImgSize=(200,200))` - 反应图像

### 指纹可视化

- `Draw.DrawMorganBit(mol, bitId, bitInfo, whichExample=0)` - 可视化 Morgan 位
- `Draw.DrawMorganBits(bits, mol, bitInfo, molsPerRow=3)` - 多个 Morgan 位
- `Draw.DrawRDKitBit(mol, bitId, bitInfo, whichExample=0)` - 可视化 RDKit 位

### IPython 集成

- `Draw.IPythonConsole` - Jupyter 集成模块
- `Draw.IPythonConsole.ipython_useSVG` - 使用 SVG (True) 或 PNG (False)
- `Draw.IPythonConsole.molSize` - 默认分子图像尺寸

### 绘图选项

- `rdMolDraw2D.MolDrawOptions()` - 获取绘图选项对象
  - `.addAtomIndices` - 显示原子索引
  - `.addBondIndices` - 显示键索引
  - `.addStereoAnnotation` - 显示立体化学
  - `.bondLineWidth` - 键线宽度
  - `.highlightBondWidthMultiplier` - 高亮宽度倍数
  - `.minFontSize` - 最小字体大小
  - `.maxFontSize` - 最大字体大小

## rdkit.Chem.rdMolDescriptors

附加描述符计算。

- `rdMolDescriptors.CalcNumRings(mol)` - 环数量
- `rdMolDescriptors.CalcNumAromaticRings(mol)` - 芳香环数量
- `rdMolDescriptors.CalcNumAliphaticRings(mol)` - 脂肪环数量
- `rdMolDescriptors.CalcNumSaturatedRings(mol)` - 饱和环数量
- `rdMolDescriptors.CalcNumHeterocycles(mol)` - 杂环数量
- `rdMolDescriptors.CalcNumAromaticHeterocycles(mol)` - 芳香杂环数量
- `rdMolDescriptors.CalcNumSpiroAtoms(mol)` - 螺原子数量
- `rdMolDescriptors.CalcNumBridgeheadAtoms(mol)` - 桥头原子数量
- `rdMolDescriptors.CalcFractionCsp3(mol)` - sp3 碳比例
- `rdMolDescriptors.CalcLabuteASA(mol)` - Labute 可及表面积
- `rdMolDescriptors.CalcTPSA(mol)` - TPSA
- `rdMolDescriptors.CalcMolFormula(mol)` - 分子式

## rdkit.Chem.Scaffolds

骨架分析。

### Murcko 骨架

- `MurckoScaffold.GetScaffoldForMol(mol)` - 获取 Murcko 骨架
- `MurckoScaffold.MakeScaffoldGeneric(mol)` - 通用骨架
- `MurckoScaffold.MurckoDecompose(mol)` - 分解为骨架和侧链

## rdkit.Chem.rdMolHash

分子哈希与标准化。

- `rdMolHash.MolHash(mol, hashFunction)` - 生成哈希
  - `rdMolHash.HashFunction.AnonymousGraph` - 匿名化结构
  - `rdMolHash.HashFunction.CanonicalSmiles` - 规范 SMILES
  - `rdMolHash.HashFunction.ElementGraph` - 元素图
  - `rdMolHash.HashFunction.MurckoScaffold` - Murcko 骨架
  - `rdMolHash.HashFunction.Regioisomer` - 区域异构体 (无立体)
  - `rdMolHash.HashFunction.NetCharge` - 净电荷
  - `rdMolHash.HashFunction.HetAtomProtomer` - 杂原子质子异构体
  - `rdMolHash.HashFunction.HetAtomTautomer` - 杂原子互变异构体

## rdkit.Chem.MolStandardize

分子标准化。

- `rdMolStandardize.Normalize(mol)` - 标准化官能团
- `rdMolStandardize.Reionize(mol)` - 修正电离状态
- `rdMolStandardize.RemoveFragments(mol)` - 移除小片段
- `rdMolStandardize.Cleanup(mol)` - 完整清理 (标准化 + 电离修正 + 移除)
- `rdMolStandardize.Uncharger()` - 创建去电荷器对象
  - `.uncharge(mol)` - 移除电荷
- `rdMolStandardize.TautomerEnumerator()` - 枚举互变异构体
  - `.Enumerate(mol)` - 生成互变异构体

- `Pairs.GetAtomPairFingerprint(mol, minLength=1, maxLength=30)` - 原子对指纹
- `Pairs.GetAtomPairFingerprintAsBitVect(mol, minLength=1, maxLength=30, nBits=2048)` - 位向量形式
- `Pairs.GetHashedAtomPairFingerprint(mol, nBits=2048, minLength=1, maxLength=30)` - 哈希版本

## rdkit.Chem.Torsions

拓扑扭转指纹。

- `Torsions.GetTopologicalTorsionFingerprint(mol, targetSize=4)` - 扭转指纹
- `Torsions.GetTopologicalTorsionFingerprintAsIntVect(mol, targetSize=4)` - 整型向量形式
- `Torsions.GetHashedTopologicalTorsionFingerprint(mol, nBits=2048, targetSize=4)` - 哈希版本

## rdkit.Chem.MACCSkeys

MACCS结构密钥。

- `MACCSkeys.GenMACCSKeys(mol)` - 生成166位MACCS密钥

## rdkit.Chem.ChemicalFeatures

药效团特征。

- `ChemicalFeatures.BuildFeatureFactory(featureFile)` - 创建特征工厂
- `factory.GetFeaturesForMol(mol)` - 获取药效团特征
- `feature.GetFamily()` - 特征族（供体、受体等）
- `feature.GetType()` - 特征类型
- `feature.GetAtomIds()` - 特征涉及原子

## rdkit.ML.Cluster.Butina

聚类算法。

- `Butina.ClusterData(distances, nPts, distThresh, isDistData=True)` - Butina聚类
  - 返回包含聚类成员的元组元组

## rdkit.Chem.rdFingerprintGenerator

现代指纹生成API（RDKit 2020.09+）。

- `rdFingerprintGenerator.GetMorganGenerator(radius=2, fpSize=2048)` - Morgan生成器
- `rdFingerprintGenerator.GetRDKitFPGenerator(minPath=1, maxPath=7, fpSize=2048)` - RDKit指纹生成器
- `rdFingerprintGenerator.GetAtomPairGenerator(minDistance=1, maxDistance=30)` - 原子对生成器
- `generator.GetFingerprint(mol)` - 生成指纹
- `generator.GetCountFingerprint(mol)` - 计数型指纹

## 通用参数

### 净化操作

- `SANITIZE_NONE` - 无净化
- `SANITIZE_ALL` - 全部操作（默认）
- `SANITIZE_CLEANUP` - 基础清理
- `SANITIZE_PROPERTIES` - 计算属性
- `SANITIZE_SYMMRINGS` - 对称化环
- `SANITIZE_KEKULIZE` - 凯库勒化芳香环
- `SANITIZE_FINDRADICALS` - 查找自由基电子
- `SANITIZE_SETAROMATICITY` - 设置芳香性
- `SANITIZE_SETCONJUGATION` - 设置共轭
- `SANITIZE_SETHYBRIDIZATION` - 设置杂化
- `SANITIZE_CLEANUPCHIRALITY` - 清理手性

### 键类型

- `BondType.SINGLE` - 单键
- `BondType.DOUBLE` - 双键
- `BondType.TRIPLE` - 三键
- `BondType.AROMATIC` - 芳香键
- `BondType.DATIVE` - 配位键
- `BondType.UNSPECIFIED` - 未指定

### 杂化类型

- `HybridizationType.S` - S
- `HybridizationType.SP` - SP
- `HybridizationType.SP2` - SP2
- `HybridizationType.SP3` - SP3
- `HybridizationType.SP3D` - SP3D
- `HybridizationType.SP3D2` - SP3D2

### 手性

- `ChiralType.CHI_UNSPECIFIED` - 未指定
- `ChiralType.CHI_TETRAHEDRAL_CW` - 顺时针
- `ChiralType.CHI_TETRAHEDRAL_CCW` - 逆时针

## 安装

```bash
# 使用 conda（推荐）
conda install -c conda-forge rdkit

# 使用 pip
pip install rdkit-pypi
```

## 导入模块

```python
# 核心功能
from rdkit import Chem
from rdkit.Chem import AllChem

# 描述符
from rdkit.Chem import Descriptors

# 绘图
from rdkit.Chem import Draw

# 相似度计算
from rdkit import DataStructs
```
