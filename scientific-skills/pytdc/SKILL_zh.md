---
name: pytdc
description: 治疗数据共享平台。提供AI就绪的药物发现数据集（ADME、毒性、DTI）、基准测试、骨架分割、分子预言器，用于治疗性机器学习和药理学预测。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# PyTDC（治疗数据共享平台）

## 概述

PyTDC 是一个开放科学平台，为药物发现与开发提供AI就绪的数据集和基准测试。该平台涵盖整个治疗流程的精选数据集，包含标准化评估指标和有意义的数据分割，分为三大类别：单实例预测（分子/蛋白质属性）、多实例预测（药物-靶点相互作用、DDI）和生成任务（分子生成、逆合成）。

## 使用场景

本技能适用于以下场景：
- 处理药物发现或治疗性机器学习数据集
- 在标准化药物任务上对机器学习模型进行基准测试
- 预测分子属性（ADME、毒性、生物活性）
- 预测药物-靶点或药物-药物相互作用
- 生成具有特定性质的新分子
- 获取带规范训练/测试分割的数据集（骨架分割、冷分割）
- 使用分子预言器进行属性优化

## 安装与配置

通过 pip 安装 PyTDC：

```bash
uv pip install PyTDC
```

升级至最新版本：

```bash
uv pip install PyTDC --upgrade
```

核心依赖（自动安装）：
- numpy, pandas, tqdm, seaborn, scikit_learn, fuzzywuzzy

特定功能所需的附加包将按需自动安装。

## 快速入门

访问任何TDC数据集的基本模式如下：

```python
from tdc.<问题域> import <任务>
data = <任务>(name='<数据集>')
split = data.get_split(method='scaffold', seed=1, frac=[0.7, 0.1, 0.2])
df = data.get_data(format='df')
```

参数说明：
- `<问题域>`：`single_pred`、`multi_pred` 或 `generation` 之一
- `<任务>`：具体任务类别（如ADME、DTI、MolGen）
- `<数据集>`：该任务下的数据集名称

**示例 - 加载ADME数据：**

```python
from tdc.single_pred import ADME
data = ADME(name='Caco2_Wang')
split = data.get_split(method='scaffold')
# 返回包含'train'、'valid'、'test'数据帧的字典
```

## 单实例预测任务

单实例预测涉及对单个生物医学实体（分子、蛋白质等）的属性进行预测。

### 可用任务类别

#### 1. ADME（吸收、分布、代谢、排泄）

预测药物分子的药代动力学属性。

```python
from tdc.single_pred import ADME
data = ADME(name='Caco2_Wang')  # 肠道渗透性
# 其他数据集：HIA_Hou, Bioavailability_Ma, Lipophilicity_AstraZeneca等
```

**常用ADME数据集：**
- Caco2 - 肠道渗透性
- HIA - 人体肠道吸收率
- Bioavailability - 口服生物利用度
- Lipophilicity - 辛醇-水分配系数
- Solubility - 水溶性
- BBB - 血脑屏障穿透性
- CYP - 细胞色素P450代谢

#### 2. 毒性（Tox）

预测化合物的毒性和副作用。

```python
from tdc.single_pred import Tox
data = Tox(name='hERG')  # 心脏毒性
# 其他数据集：AMES, DILI, Carcinogens_Lagunin等
```

**常用毒性数据集：**
- hERG - 心脏毒性
- AMES - 致突变性
- DILI - 药物性肝损伤
- Carcinogens - 致癌性
- ClinTox - 临床试验毒性

#### 3. HTS（高通量筛选）

基于筛选数据的生物活性预测。

```python
from tdc.single_pred import HTS
data = HTS(name='SARSCoV2_Vitro_Touret')
```

#### 4. QM（量子力学）

分子的量子力学属性。

```python
from tdc.single_pred import QM
data = QM(name='QM7')
```

#### 5. 其他单预测任务

- **Yields**：化学反应产率预测
- **Epitope**：生物制剂表位预测
- **Develop**：开发阶段预测
- **CRISPROutcome**：基因编辑结果预测

### 数据格式

单预测数据集通常返回包含以下列的数据帧：
- `Drug_ID` 或 `Compound_ID`：唯一标识符
- `Drug` 或 `X`：SMILES字符串或分子表示
- `Y`：目标标签（连续值或二分类）

## 多实例预测任务

多实例预测涉及对多个生物医学实体间相互作用的属性进行预测。

### 可用任务类别

#### 1. DTI（药物-靶点相互作用）

预测药物与蛋白质靶点间的结合亲和力。

```python
from tdc.multi_pred import DTI
data = DTI(name='BindingDB_Kd')
split = data.get_split()
```

**可用数据集：**
- BindingDB_Kd - 解离常数（52,284对）
- BindingDB_IC50 - 半抑制浓度（991,486对）
- BindingDB_Ki - 抑制常数（375,032对）
- DAVIS, KIBA - 激酶结合数据集

**数据格式：** Drug_ID, Target_ID, Drug (SMILES), Target (序列), Y (结合亲和力)

#### 2. DDI（药物-药物相互作用）

预测药物对间的相互作用。

```python
from tdc.multi_pred import DDI
data = DDI(name='DrugBank')
split = data.get_split()
```

多分类任务预测相互作用类型。数据集包含191,808个DDI对，涉及1,706种药物。

#### 3. PPI（蛋白质-蛋白质相互作用）

预测蛋白质间相互作用。

```python
from tdc.multi_pred import PPI
data = PPI(name='HuRI')
```

#### 4. 其他多预测任务

- **GDA**：基因-疾病关联
- **DrugRes**：药物抗性预测
- **DrugSyn**：药物协同作用预测
- **PeptideMHC**：肽-MHC结合
- **AntibodyAff**：抗体亲和力预测
- **MTI**：miRNA-靶点相互作用
- **Catalyst**：催化剂预测
- **TrialOutcome**：临床试验结果预测

## 生成任务

生成任务涉及创建具有特定属性的新型生物医学实体。

### 1. 分子生成（MolGen）

生成具有理想化学性质的多样化新分子。

```python
from tdc.generation import MolGen
data = MolGen(name='ChEMBL_V29')
split = data.get_split()
```

结合预言器优化特定属性：

```python
from tdc import Oracle
oracle = Oracle(name='GSK3B')
score = oracle('CC(C)Cc1ccc(cc1)C(C)C(O)=O')  # 评估SMILES
```

所有可用预言器函数参见 `references/oracles.md`。

### 2. 逆合成（RetroSyn）

预测合成目标分子所需的反应物。

```python
from tdc.generation import RetroSyn
data = RetroSyn(name='USPTO')
split = data.get_split()
```

数据集包含来自USPTO数据库的1,939,253个反应。

### 3. 配对分子生成

生成分子对（例如前药-药物对）。

```python
from tdc.generation import PairMolGen
data = PairMolGen(name='Prodrug')
```

详细预言器文档和分子生成流程参见 `references/oracles.md` 和 `scripts/molecular_generation.py`。

## 基准测试组

基准测试组提供相关数据集的精选集合，用于系统性模型评估。

### ADMET基准测试组

```python
from tdc.benchmark_group import admet_group
group = admet_group(path='data/')

# 获取基准数据集
benchmark = group.get('Caco2_Wang')
predictions = {}

for seed in [1, 2, 3, 4, 5]:
    train, valid = benchmark['train'], benchmark['valid']
    # 在此训练模型
    predictions[seed] = model.predict(benchmark['test'])

# 使用要求的5个种子进行评估
results = group.evaluate(predictions)
```

**ADMET组包含22个数据集**，覆盖吸收、分布、代谢、排泄和毒性。

### 其他基准测试组

可用基准测试组包括：
- ADMET属性
- 药物-靶点相互作用
- 药物组合预测
- 其他专业治疗任务

基准评估流程参见 `scripts/benchmark_evaluation.py`。

## 数据功能

TDC提供四类全面的数据处理工具。

### 1. 数据集分割

通过多种策略获取训练/验证/测试分区：

```python
# 骨架分割（多数任务默认）
split = data.get_split(method='scaffold', seed=1, frac=[0.7, 0.1, 0.2])

# 随机分割
split = data.get_split(method='random', seed=42, frac=[0.8, 0.1, 0.1])

# 冷分割（用于DTI/DDI任务）
split = data.get_split(method='cold_drug', seed=1)  # 测试集含未见药物
split = data.get_split(method='cold_target', seed=1)  # 测试集含未见靶点
```

**可用分割策略：**
- `random`：随机洗牌
- `scaffold`：基于骨架（保障化学多样性）
- `cold_drug`、`cold_target`、`cold_drug_target`：用于DTI任务
- `temporal`：时间序列数据集的时间分割

### 2. 模型评估

使用标准化指标进行评估：

```python
from tdc import Evaluator

# 二分类任务
evaluator = Evaluator(name='ROC-AUC')
score = evaluator(y_true, y_pred)

# 回归任务
evaluator = Evaluator(name='RMSE')
score = evaluator(y_true, y_pred)
```

**可用指标：** ROC-AUC、PR-AUC、F1、准确率、RMSE、MAE、R2、Spearman、Pearson等。

### 3. 数据处理

TDC提供11项核心处理工具：

```python
from tdc.chem_utils import MolConvert

# 分子格式转换
converter = MolConvert(src='SMILES', dst='PyG')
pyg_graph = converter('CC(C)Cc1ccc(cc1)C(C)C(O)=O')
```

**处理工具包括：**
- 分子格式转换（SMILES、SELFIES、PyG、DGL、ECFP等）
- 分子过滤器（PAINS、类药性）
- 标签二值化与单位转换
- 数据平衡（过采样/欠采样）
- 配对数据负采样
- 图结构转换
- 实体检索（CID转SMILES、UniProt转序列）

完整工具文档参见 `references/utilities.md`。

### 4. 分子生成预言器

TDC提供17+个分子优化预言器：

```python
from tdc import Oracle

# 单预言器
oracle = Oracle(name='DRD2')
score = oracle('CC(C)Cc1ccc(cc1)C(C)C(O)=O')

# 多预言器
oracle = Oracle(name='JNK3')
scores = oracle(['SMILES1', 'SMILES2', 'SMILES3'])
```

完整预言器文档参见 `references/oracles.md`。

## 高级功能

### 检索可用数据集

```python
from tdc.utils import retrieve_dataset_names

# 获取所有ADME数据集
adme_datasets = retrieve_dataset_names('ADME')

# 获取所有DTI数据集
dti_datasets = retrieve_dataset_names('DTI')
```

### 标签转换

```python
# 获取标签映射
label_map = data.get_label_map(name='DrugBank')

# 转换标签
from tdc.chem_utils import label_transform
transformed = label_transform(y, from_unit='nM', to_unit='p')
```

### 数据库查询

```python
from tdc.utils import cid2smiles, uniprot2seq

# PubChem CID转SMILES
smiles = cid2smiles(2244)

# UniProt ID转氨基酸序列
sequence = uniprot2seq('P12345')
```

## 常用工作流

### 工作流1：训练单预测模型

完整示例参见 `scripts/load_and_split_data.py`：

```python
from tdc.single_pred import ADME
from tdc import Evaluator

# 加载数据
data = ADME(name='Caco2_Wang')
split = data.get_split(method='scaffold', seed=42)

train, valid, test = split['train'], split['valid'], split['test']

# 训练模型（用户实现）
# model.fit(train['Drug'], train['Y'])

# 评估
evaluator = Evaluator(name='MAE')
# score = evaluator(test['Y'], predictions)
```

### 工作流2：基准测试评估

完整示例参见 `scripts/benchmark_evaluation.py`，包含多种子和规范评估协议。

### 工作流3：基于预言器的分子生成

参见 `scripts/molecular_generation.py`，展示使用预言器进行目标导向生成的示例。

## 资源

本技能包含TDC常用工作流的捆绑资源：

### scripts/

- `load_and_split_data.py`：使用多种策略加载和分割TDC数据集的模板
- `benchmark_evaluation.py`：执行基准测试组评估的模板（含5种子规范协议）
- `molecular_generation.py`：使用预言器进行分子生成的模板

### references/

- `datasets.md`：按任务类型组织的完整数据集目录
- `oracles.md`：全部17+分子生成预言器的完整文档
- `utilities.md`：数据处理、分割和评估工具的详细指南

## 附加资源

- **官方网站**：https://tdcommons.ai
- **文档**：https://tdc.readthedocs.io
- **GitHub**：https://github.com/mims-harvard/TDC
- **论文**：NeurIPS 2021 -《治疗数据共享平台：药物发现与开发的机器学习数据集和任务》
