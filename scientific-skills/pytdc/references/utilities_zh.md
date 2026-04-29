# TDC 实用工具与数据函数

本文档提供 TDC 数据处理、评估和实用函数的全面文档。

## 概述

TDC 提供的实用工具分为四大类别：
1. **数据集划分** - 训练/验证/测试分区策略
2. **模型评估** - 标准化性能指标
3. **数据处理** - 分子转换、过滤与变换
4. **实体检索** - 数据库查询与转换

## 1. 数据集划分

数据集划分对评估模型泛化能力至关重要。TDC 提供专为治疗性机器学习设计的多种划分策略。

### 基础划分用法

```python
from tdc.single_pred import ADME

data = ADME(name='Caco2_Wang')

# 使用默认参数获取划分
split = data.get_split()
# 返回: {'train': DataFrame, 'valid': DataFrame, 'test': DataFrame}

# 自定义划分参数
split = data.get_split(
    method='scaffold',
    seed=42,
    frac=[0.7, 0.1, 0.2]
)
```

### 划分方法

#### 随机划分
数据随机打乱 - 适用于通用机器学习任务。

```python
split = data.get_split(method='random', seed=1)
```

**适用场景:**
- 基线模型评估
- 化学/时间结构不重要时
- 快速原型设计

**不推荐场景:**
- 真实药物发现场景
- 评估新化学物质的泛化能力

#### 骨架划分
基于分子骨架（Bemis-Murcko 骨架）划分 - 确保测试分子在结构上与训练集不同。

```python
split = data.get_split(method='scaffold', seed=1)
```

**适用场景:**
- 多数单预测任务的默认方法
- 评估新化学系列的泛化能力
- 真实药物发现场景

**工作原理:**
1. 从每个分子提取 Bemis-Murcko 骨架
2. 按骨架分组分子
3. 将骨架分配到训练/验证/测试集
4. 确保测试分子具有未见过的骨架

#### 冷划分（DTI/DDI 任务）
针对多实例预测，冷划分确保测试集包含未见过的药物、靶点或两者。

**冷药物划分:**
```python
from tdc.multi_pred import DTI
data = DTI(name='BindingDB_Kd')
split = data.get_split(method='cold_drug', seed=1)
```
- 测试集包含训练中未见的药物
- 评估对新化合物的泛化能力

**冷靶点划分:**
```python
split = data.get_split(method='cold_target', seed=1)
```
- 测试集包含训练中未见的靶点
- 评估对新蛋白质的泛化能力

**冷药物-靶点划分:**
```python
split = data.get_split(method='cold_drug_target', seed=1)
```
- 测试集包含全新药物-靶点对
- 最具挑战性的评估场景

#### 时间划分
适用于含时间信息的数据集 - 确保测试数据来自更晚时间点。

```python
split = data.get_split(method='temporal', seed=1)
```

**适用场景:**
- 带时间戳的数据集
- 模拟前瞻性预测
- 临床试验结果预测

### 自定义划分比例

```python
# 80% 训练, 10% 验证, 10% 测试
split = data.get_split(method='scaffold', frac=[0.8, 0.1, 0.1])

# 70% 训练, 15% 验证, 15% 测试
split = data.get_split(method='scaffold', frac=[0.7, 0.15, 0.15])
```

### 分层划分

针对标签不平衡的分类任务：

```python
split = data.get_split(method='scaffold', stratified=True)
```

在训练/验证/测试集中保持标签分布一致性。

## 2. 模型评估

TDC 为不同任务类型提供标准化评估指标。

### 基础评估器用法

```python
from tdc import Evaluator

# 初始化评估器
evaluator = Evaluator(name='ROC-AUC')

# 评估预测结果
score = evaluator(y_true, y_pred)
```

### 分类指标

#### ROC-AUC
受试者工作特征曲线下面积

```python
evaluator = Evaluator(name='ROC-AUC')
score = evaluator(y_true, y_pred_proba)
```

**最佳适用:**
- 二分类问题
- 不平衡数据集
- 整体判别能力评估

**范围:** 0-1（越高越好，0.5 表示随机）

#### PR-AUC
精确率-召回率曲线下面积

```python
evaluator = Evaluator(name='PR-AUC')
score = evaluator(y_true, y_pred_proba)
```

**最佳适用:**
- 高度不平衡数据集
- 正样本稀少时
- 作为 ROC-AUC 的补充

**范围:** 0-1（越高越好）

#### F1 分数
精确率与召回率的调和平均数

```python
evaluator = Evaluator(name='F1')
score = evaluator(y_true, y_pred_binary)
```

**最佳适用:**
- 精确率与召回率的平衡
- 多分类问题

**范围:** 0-1（越高越好）

#### 准确率
正确预测的比例

```python
evaluator = Evaluator(name='Accuracy')
score = evaluator(y_true, y_pred_binary)
```

**最佳适用:**
- 平衡数据集
- 简单基线指标

**不推荐用于:** 不平衡数据集

#### Cohen's Kappa
考虑随机因素的预测与真实标签一致性

```python
evaluator = Evaluator(name='Kappa')
score = evaluator(y_true, y_pred_binary)
```

**范围:** -1 到 1（越高越好，0 表示随机）

### 回归指标

#### RMSE - 均方根误差
```python
evaluator = Evaluator(name='RMSE')
score = evaluator(y_true, y_pred)
```

**最佳适用:**
- 连续值预测
- 对大幅误差惩罚更重

**范围:** 0-∞（越低越好）

#### MAE - 平均绝对误差
```python
evaluator = Evaluator(name='MAE')
score = evaluator(y_true, y_pred)
```

**最佳适用:**
- 连续值预测
- 比 RMSE 对异常值更鲁棒

**范围:** 0-∞（越低越好）

#### R² - 决定系数
```python
evaluator = Evaluator(name='R2')
score = evaluator(y_true, y_pred)
```

**最佳适用:**
- 模型解释的方差
- 不同模型比较

**范围:** -∞ 到 1（越高越好，1 表示完美）

#### MSE - 均方误差
```python
evaluator = Evaluator(name='MSE')
score = evaluator(y_true, y_pred)
```

**范围:** 0-∞（越低越好）

### 排序指标

#### Spearman 相关系数
秩相关系数

```python
evaluator = Evaluator(name='Spearman')
score = evaluator(y_true, y_pred)
```

**最佳适用:**
- 排序任务
- 非线性关系
- 序数数据

**范围:** -1 到 1（越高越好）

#### Pearson 相关系数
线性相关系数

```python
evaluator = Evaluator(name='Pearson')
score = evaluator(y_true, y_pred)
```

**最佳适用:**
- 线性关系
- 连续数据

**范围:** -1 到 1（越高越好）

### 多标签分类

```python
evaluator = Evaluator(name='Micro-F1')
score = evaluator(y_true_multilabel, y_pred_multilabel)
```

可用指标: `Micro-F1`, `Macro-F1`, `Micro-AUPR`, `Macro-AUPR`

### 基准组评估

基准组评估需要多个随机种子：

```python
from tdc.benchmark_group import admet_group

group = admet_group(path='data/')
benchmark = group.get('Caco2_Wang')

# 预测结果需为以种子为键的字典
predictions = {}
for seed in [1, 2, 3, 4, 5]:
    # 训练模型并预测
    predictions[seed] = model_predictions

# 跨种子计算均值与标准差
results = group.evaluate(predictions)
print(results)  # {'Caco2_Wang': [mean_score, std_score]}
```

## 3. 数据处理

TDC 提供 11 项全面的数据处理工具。

### 分子格式转换

支持约 15 种分子表示间的转换。

```python
from tdc.chem_utils import MolConvert

# SMILES 转 PyTorch Geometric
converter = MolConvert(src='SMILES', dst='PyG')
pyg_graph = converter('CC(C)Cc1ccc(cc1)C(C)C(O)=O')

# SMILES 转 DGL
converter = MolConvert(src='SMILES', dst='DGL')
dgl_graph = converter('CC(C)Cc1ccc(cc1)C(C)C(O)=O')

# SMILES 转摩根指纹 (ECFP)
converter = MolConvert(src='SMILES', dst='ECFP')
fingerprint = converter('CC(C)Cc1ccc(cc1)C(C)C(O)=O')
```

**可用格式:**
- **文本**: SMILES, SELFIES, InChI
- **指纹**: ECFP (Morgan), MACCS, RDKit, AtomPair, TopologicalTorsion
- **图结构**: PyG (PyTorch Geometric), DGL (Deep Graph Library)
- **3D结构**: Graph3D, Coulomb Matrix, Distance Matrix

**批量转换:**
```python
converter = MolConvert(src='SMILES', dst='PyG')
graphs = converter(['SMILES1', 'SMILES2', 'SMILES3'])
```

### 分子过滤器

使用精选化学规则过滤非类药分子。

```python
from tdc.chem_utils import MolFilter

# 初始化带规则的过滤器
mol_filter = MolFilter(
    rules=['PAINS', 'BMS'],  # 化学过滤规则
    property_filters_dict={
        'MW': (150, 500),      # 分子量范围
        'LogP': (-0.4, 5.6),   # 亲脂性范围
        'HBD': (0, 5),         # 氢键供体数
        'HBA': (0, 10)         # 氢键受体数
    }
)

# 过滤分子
filtered_smiles = mol_filter(smiles_list)
```

**可用过滤规则:**
- `PAINS` - 泛分析干扰化合物
- `BMS` - 百时美施贵宝 HTS 筛选规则
- `Glaxo` - 葛兰素史克过滤规则
- `Dundee` - 邓迪大学过滤规则
- `Inpharmatica` - Inpharmatica 过滤规则
- `LINT` - 辉瑞 LINT 过滤规则

### 标签分布可视化

```python
# 可视化标签分布
data.label_distribution()

# 打印统计信息
data.print_stats()
```

显示直方图并计算连续标签的均值、中位数和标准差。

### 标签二值化

通过阈值将连续标签转为二值。

```python
from tdc.utils import binarize

# 按阈值二值化
binary_labels = binarize(y_continuous, threshold=5.0, order='ascending')
# order='ascending': 值≥阈值转为1
# order='descending': 值≤阈值转为1
```

### 标签单位转换

不同测量单位间的转换。

```python
from tdc.chem_utils import label_transform

# nM 转 pKd
y_pkd = label_transform(y_nM, from_unit='nM', to_unit='p')

# μM 转 nM
y_nM = label_transform(y_uM, from_unit='uM', to_unit='nM')
```

**可用转换:**
- 结合亲和力: nM, μM, pKd, pKi, pIC50
- 对数转换
- 自然对数转换

### 标签含义

获取标签的可解释描述。

```python
# 获取标签映射
label_map = data.get_label_map(name='DrugBank')
print(label_map)
# {0: '无相互作用', 1: '效应增强', 2: '效应减弱', ...}
```

### 数据平衡

通过过采样/欠采样处理类别不平衡。

```python
from tdc.utils import balance

# 过采样少数类
X_balanced, y_balanced = balance(X, y, method='oversample')

# 欠采样多数类
X_balanced, y_balanced = balance(X, y, method='undersample')
```

### 配对数据图转换

将配对数据转换为图表示。

```python
from tdc.utils import create_graph_from_pairs

# 从药物-药物对创建图
graph = create_graph_from_pairs(
    pairs=ddi_pairs,  # [(药物1, 药物2, 标签), ...]
    format='edge_list'  # 或 'PyG', 'DGL'
)
```

### 负样本生成

为二分类任务生成负样本。

```python
from tdc.utils import negative_sample

# 为 DTI 生成负样本
negative_pairs = negative_sample(
    positive_pairs=known_interactions,
    all_drugs=drug_list,
    all_targets=target_list,
    ratio=1.0  # 负:正样本比例
)
```

**应用场景:**
- 药物-靶点相互作用预测
- 药物-药物相互作用任务
- 创建平衡数据集

### 实体检索

数据库标识符间的转换。

#### PubChem CID 转 SMILES
```python
from tdc.utils import cid2smiles

smiles = cid2smiles(2244)  # 阿司匹林
# 返回: 'CC(=O)Oc1ccccc1C(=O)O'
```

#### UniProt ID 转氨基酸序列
```python
from tdc.utils import uniprot2seq

sequence = uniprot2seq('P12345')
# 返回: 'MVKVYAPASS...'
```

#### 批量检索
```python
# 多个 CID
smiles_list = [cid2smiles(cid) for cid in [2244, 5090, 6323]]

# 多个 UniProt ID
sequences = [uniprot2seq(uid) for uid in ['P12345', 'Q9Y5S9']]
```

## 4. 高级工具

### 检索数据集名称

```python
from tdc.utils import retrieve_dataset_names

# 获取任务的所有数据集
adme_datasets = retrieve_dataset_names('ADME')
dti_datasets = retrieve_dataset_names('DTI')
tox_datasets = retrieve_dataset_names('Tox')

print(f"ADME 数据集: {adme_datasets}")
```

### 模糊搜索

TDC 支持数据集名称的模糊匹配：

```python
from tdc.single_pred import ADME

# 以下均有效（容错匹配）
data = ADME(name='Caco2_Wang')
data = ADME(name='caco2_wang')
data = ADME(name='Caco2')  # 部分匹配
```

### 数据格式选项

```python
# Pandas DataFrame (默认)
df = data.get_data(format='df')

# 字典格式
data_dict = data.get_data(format='dict')

# DeepPurpose 格式（用于 DeepPurpose 库）
dp_format = data.get_data(format='DeepPurpose')

# PyG/DGL 图结构（如适用）
graphs = data.get_data(format='PyG')
```

### 数据加载工具

```python
from tdc.utils import create_fold

# 创建交叉验证折
folds = create_fold(data, fold=5, seed=42)
# 返回 (train_idx, test_idx) 元组列表

# 遍历各折
for i, (train_idx, test_idx) in enumerate(folds):
    train_data = data.iloc[train_idx]
    test_data = data.iloc[test_idx]
    # 训练与评估
```

## 常用工作流

### 工作流 1: 完整数据处理流程

```python
from tdc.single_pred import ADME
from tdc import Evaluator
from tdc.chem_utils import MolConvert, MolFilter

# 1. 加载数据
data = ADME(name='Caco2_Wang')

# 2. 过滤分子
mol_filter = MolFilter(rules=['PAINS'])
filtered_data = data.get_data()
filtered_data = filtered_data[
    filtered_data['Drug'].apply(lambda x: mol_filter([x]))
]

# 3. 划分数据
split = data.get_split(method='scaffold', seed=42)
train, valid, test = split['train'], split['valid'], split['test']

# 4. 转换为图表示
converter = MolConvert(src='SMILES', dst='PyG')
train_graphs = converter(train['Drug'].tolist())

# 5. 训练模型（用户实现）
# model.fit(train_graphs, train

# score = evaluator(test['Y'], predictions)
```

### 工作流程 2：多任务学习准备

```python
from tdc.benchmark_group import admet_group
from tdc.chem_utils import MolConvert

# 加载基准测试组
group = admet_group(path='data/')

# 获取多个数据集
datasets = ['Caco2_Wang', 'HIA_Hou', 'Bioavailability_Ma']
all_data = {}

for dataset_name in datasets:
    benchmark = group.get(dataset_name)
    all_data[dataset_name] = benchmark

# 多任务学习准备
converter = MolConvert(src='SMILES', dst='ECFP')
# 处理每个数据集...
```

### 工作流程 3：DTI冷分离评估

```python
from tdc.multi_pred import DTI
from tdc import Evaluator

# 加载DTI数据
data = DTI(name='BindingDB_Kd')

# 冷药物分离
split = data.get_split(method='cold_drug', seed=42)
train, test = split['train'], split['test']

# 验证无药物重叠
train_drugs = set(train['Drug_ID'])
test_drugs = set(test['Drug_ID'])
assert len(train_drugs & test_drugs) == 0, "检测到药物泄漏！"

# 训练与评估
# model.fit(train)
evaluator = Evaluator(name='RMSE')
# score = evaluator(test['Y'], predictions)
```

## 最佳实践

1. **始终使用有意义的数据分离** - 使用骨架分离或冷分离实现真实评估
2. **多随机种子验证** - 使用多个随机种子运行实验确保结果稳健
3. **选择合适指标** - 选用与任务及数据集特性匹配的评估指标
4. **数据过滤** - 训练前移除PAINS和非类药分子
5. **格式转换** - 将分子转换为模型适用的格式
6. **批处理** - 大型数据集使用批量操作提升效率

## 性能优化建议

- 使用批量模式转换分子以加速处理
- 缓存转换后的分子表示避免重复计算
- 为框架选用合适数据格式（PyG、DGL等）
- 在流程早期过滤数据以减少计算量

## 参考文献

- TDC文档：https://tdc.readthedocs.io
- 数据功能：https://tdcommons.ai/fct_overview/
- 评估指标：https://tdcommons.ai/functions/model_eval/
- 数据分离方法：https://tdcommons.ai/functions/data_split/
