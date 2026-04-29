# Molfeat 中可用的特征化器

本文档按类别整理，全面收录了 molfeat 中所有可用的特征化器。

## 基于 Transformer 的语言模型

使用 SMILES/SELFIES 表示生成分子嵌入的预训练 Transformer 模型。

### RoBERTa 风格模型
- **Roberta-Zinc480M-102M** - 基于 ZINC 数据库约 4.8 亿个 SMILES 字符串训练的 RoBERTa 掩码语言模型
- **ChemBERTa-77M-MLM** - 基于 RoBERTa 架构，在 7700 万 PubChem 化合物上训练的掩码语言模型
- **ChemBERTa-77M-MTR** - 在 PubChem 化合物上训练的多任务回归版本

### GPT 风格自回归模型
- **GPT2-Zinc480M-87M** - 基于 ZINC 约 4.8 亿 SMILES 训练的 GPT-2 自回归语言模型
- **ChemGPT-1.2B** - 在 PubChem10M 上预训练的大型 Transformer（12 亿参数）
- **ChemGPT-19M** - 在 PubChem10M 上预训练的中型 Transformer（1900 万参数）
- **ChemGPT-4.7M** - 在 PubChem10M 上预训练的小型 Transformer（470 万参数）

### 专用 Transformer 模型
- **MolT5** - 用于分子描述和文本生成的自监督框架

## 图神经网络 (GNN)

基于分子图结构操作的预训练图神经网络模型。

### GIN（图同构网络）变体
所有模型均在 ChEMBL 分子上预训练，具有不同目标：
- **gin-supervised-masking** - 节点掩码监督目标
- **gin-supervised-infomax** - 图级互信息最大化监督目标
- **gin-supervised-edgepred** - 边预测监督目标
- **gin-supervised-contextpred** - 上下文预测监督目标

### 其他基于图的模型
- **JTVAE_zinc_no_kl** - 用于分子生成的连接树 VAE（在 ZINC 上训练）
- **Graphormer-pcqm4mv2** - 在 PCQM4Mv2 量子化学数据集上预训练的图 Transformer，用于 HOMO-LUMO 能隙预测

## 分子描述符

用于计算物理化学性质和分子特征的计算器。

### 2D 描述符
- **desc2D** / **rdkit2D** - 200+ 个 RDKit 二维分子描述符，包括：
  - 分子量、logP、TPSA
  - 氢键供体/受体
  - 可旋转键
  - 环计数与芳香性
  - 分子复杂度指标

### 3D 描述符
- **desc3D** / **rdkit3D** - RDKit 三维分子描述符（需构象生成）
  - 惯性矩
  - PMI（主惯性矩）比率
  - 非球面性、偏心率
  - 回转半径

### 综合描述符集
- **mordred** - 超过 1800 个分子描述符，涵盖：
  - 组成描述符
  - 拓扑指数
  - 连接性指数
  - 信息含量
  - 2D/3D 自相关
  - WHIM 描述符
  - GETAWAY 描述符
  - 以及更多

### 电拓扑描述符
- **estate** - 电拓扑状态（E-State）指数，编码：
  - 原子环境信息
  - 电子与拓扑性质
  - 杂原子贡献

## 分子指纹

表示分子子结构的二进制或基于计数的定长向量。

### 圆形指纹（ECFP 风格）
- **ecfp** / **ecfp:2** / **ecfp:4** / **ecfp:6** - 扩展连通性指纹
  - 半径变体（2/4/6 对应直径）
  - 默认：半径=3，2048 位
  - 相似性搜索最常用
- **ecfp-count** - ECFP 计数版本（非二进制）
- **fcfp** / **fcfp-count** - 功能类圆形指纹
  - 类似 ECFP 但使用功能基团
  - 更适合基于药效团的相似性

### 基于路径的指纹
- **rdkit** - 基于线性路径的 RDKit 拓扑指纹
- **pattern** - 模式指纹（类似 MACCS 但自动化生成）
- **layered** - 包含多子结构层的分层指纹

### 基于键的指纹
- **maccs** - MACCS 键（166 位结构键）
  - 预定义子结构固定集
  - 适用于骨架跃迁
  - 计算快速
- **avalon** - Avalon 指纹
  - 类似 MACCS 但特征更丰富
  - 针对相似性搜索优化

### 原子对指纹
- **atompair** - 原子对指纹
  - 编码原子对及其间距
  - 适用于 3D 相似性
- **atompair-count** - 原子对计数版本

### 拓扑扭转指纹
- **topological** - 拓扑扭转指纹
  - 编码 4 个连接原子的序列
  - 捕获局部拓扑结构
- **topological-count** - 拓扑扭转计数版本

### 最小哈希指纹
- **map4** - 4 键内最小哈希原子对指纹
  - 结合原子对与 ECFP 概念
  - 默认：1024 维
  - 适用于大规模数据集的高效计算
- **secfp** - SMILES 扩展连通性指纹
  - 直接在 SMILES 字符串操作
  - 同时捕获子结构和原子对信息

### 扩展简化图
- **erg** - 扩展简化图
  - 使用药效团点替代原子
  - 在保留关键特征的同时降低图复杂度

## 药效团描述符

基于药理学相关功能基团及其空间关系的特征。

### CATS（化学高级模板搜索）
- **cats2D** - 二维 CATS 描述符
  - 药效团点对分布
  - 基于最短路径的距离
  - 默认 21 个描述符
- **cats3D** - 三维 CATS 描述符
  - 基于欧几里得距离
  - 需构象生成
- **cats2D_pharm** / **cats3D_pharm** - 药效团变体

### Gobbi 药效团
- **gobbi2D** - 二维药效团指纹
  - 8 种药效团特征类型：
    - 疏水性
    - 芳香性
    - 氢键受体
    - 氢键供体
    - 可电离正电荷
    - 可电离负电荷
    - 聚合疏水基团
  - 适用于虚拟筛选

### Pmapper 药效团
- **pmapper2D** - 二维药效团特征谱
- **pmapper3D** - 三维药效团特征谱
  - 高维药效团描述符
  - 适用于 QSAR 和相似性搜索

## 形状描述符

捕获 3D 分子形状和静电性质的描述符。

### USR（超快形状识别）
- **usr** - 基础 USR 描述符
  - 12 维形状分布编码
  - 计算极快
- **usrcat** - 带药效团约束的 USR
  - 60 维（每特征类型 12 维）
  - 结合形状与药效团信息

### 静电形状
- **electroshape** - 静电形状描述符
  - 结合分子形状、手性和静电特性
  - 适用于蛋白质-配体对接预测

## 基于骨架的描述符

基于分子骨架和核心结构的描述符。

### 骨架键
- **scaffoldkeys** - 骨架键计算器
  - 40+ 骨架特性
  - 生物电子等排骨架表示
  - 捕获核心结构特征

## 用于 GNN 输入的图特征化器

用于构建图神经网络图表示的原子级和键级特征。

### 原子级特征
- **atom-onehot** - 独热编码原子特征
- **atom-default** - 默认原子特征化，包括：
  - 原子序数
  - 度、形式电荷
  - 杂化状态
  - 芳香性
  - 氢原子数量

### 键级特征
- **bond-onehot** - 独热编码键特征
- **bond-default** - 默认键特征化，包括：
  - 键类型（单键、双键、三键、芳香键）
  - 共轭性
  - 环成员关系
  - 立体化学

## 集成预训练模型集合

Molfeat 整合了多种来源的模型：

### HuggingFace 模型
通过 HuggingFace hub 访问 Transformer 模型：
- ChemBERTa 变体
- ChemGPT 变体
- MolT5
- 自定义上传模型

### DGL-LifeSci 模型
来自 DGL-Life 的预训练 GNN 模型：
- 不同预训练任务的 GIN 变体
- AttentiveFP 模型
- MPNN 模型

### FCD（Fréchet ChemNet 距离）
- **fcd** - 用于分子生成评估的预训练 CNN

### Graphormer 模型
- 微软研究院的图 Transformer
- 在量子化学数据集上预训练

## 使用说明

### 选择特征化器

**对于传统机器学习（随机森林、SVM 等）：**
- 从 **ecfp** 或 **maccs** 指纹开始
- 尝试 **desc2D** 获取可解释模型
- 使用 **FeatConcat** 组合多个指纹

**对于深度学习：**
- 使用 **ChemBERTa** 或 **ChemGPT** 获取 Transformer 嵌入
- 使用 **gin-supervised-*** 获取图神经网络嵌入
- 量子性质预测可考虑 **Graphormer**

**对于相似性搜索：**
- **ecfp** - 通用型，最流行
- **maccs** - 快速，适用于骨架跃迁
- **map4** - 大规模搜索高效
- **usr** / **usrcat** - 3D 形状相似性

**对于基于药效团的方法：**
- **fcfp** - 基于功能基团
- **cats2D/3D** - 药效团点对分布
- **gobbi2D** - 显式药效团特征

**对于可解释性：**
- **desc2D** / **mordred** - 具名描述符
- **maccs** - 可解释子结构键
- **scaffoldkeys** - 基于骨架的特征

### 模型依赖项

部分特征化器需要额外依赖：

- **DGL 模型** (gin-*, jtvae)：`pip install "molfeat[dgl]"`
- **Graphormer**：`pip install "molfeat[graphormer]"`
- **Transformers** (ChemBERTa, ChemGPT, MolT5)：`pip install "molfeat[transformer]"`
- **FCD**：`pip install "molfeat[fcd]"`
- **MAP4**：`pip install "molfeat[map4]"`
- **所有依赖**：`pip install "molfeat[all]"`

### 访问所有可用模型

```python
from molfeat.store.modelstore import ModelStore

store = ModelStore()
all_models = store.available_models

# 打印所有可用特征化器
for model in all_models:
    print(f"{model.name}: {model.description}")

# 搜索特定类型
transformers = [m for m in all_models if "transformer" in m.tags]
gnn_models = [m for m in all_models if "gnn" in m.tags]
fingerprints = [m for m in all_models if "fingerprint" in m.tags]
```

## 性能特征

### 计算速度（相对）
**最快：**
- maccs
- ecfp
- rdkit 指纹
- usr

**中等：**
- desc2D
- cats2D
- 多数指纹

**较慢：**
- mordred（1800+ 描述符）
- desc3D（需构象生成）
- 一般 3D 描述符

**最慢（首次运行）：**
- 预训练模型（ChemBERTa, ChemGPT, GIN）
- 注：后续运行受益于缓存

### 维度

**低维（< 200 维）：**
- maccs (167)
- usr (12)
- usrcat (60)

**中维（200-2000 维）：**
- desc2D (~200)
- ecfp (默认 2048，可配置)
- map4 (默认 1024)

**高维（> 2000 维）：**
- mordred (1800+)
- 组合指纹
- 部分 Transformer 嵌入

**可变维度：**
- Transformer 模型（通常 768-1024）
- GNN 模型（取决于架构）
