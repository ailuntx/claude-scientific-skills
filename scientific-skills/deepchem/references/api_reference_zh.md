# DeepChem API 参考文档

本文档提供 DeepChem 核心 API 的全面参考，按功能模块组织。

## 数据处理

### 数据加载器

#### 文件格式加载器
- **CSVLoader**：从 CSV 文件加载表格数据，支持自定义特征处理
- **UserCSVLoader**：用户自定义 CSV 加载，支持灵活列配置
- **SDFLoader**：处理分子结构文件（SDF 格式）
- **JsonLoader**：导入 JSON 结构数据集
- **ImageLoader**：为计算机视觉任务加载图像数据

#### 生物数据加载器
- **FASTALoader**：处理 FASTA 格式的蛋白质/DNA 序列
- **FASTQLoader**：处理带质量分数的 FASTQ 测序数据
- **SAMLoader/BAMLoader/CRAMLoader**：支持序列比对格式

#### 专用加载器
- **DFTYamlLoader**：处理密度泛函理论计算数据
- **InMemoryLoader**：直接从 Python 对象加载数据

### 数据集类

- **NumpyDataset**：封装 NumPy 数组用于内存数据处理
- **DiskDataset**：管理磁盘存储的大型数据集，降低内存开销
- **ImageDataset**：面向图像机器学习任务的专用容器

### 数据拆分器

#### 通用拆分器
- **RandomSplitter**：随机数据集划分
- **IndexSplitter**：按指定索引拆分
- **SpecifiedSplitter**：使用预定义拆分
- **RandomStratifiedSplitter**：分层随机拆分
- **SingletaskStratifiedSplitter**：单任务分层拆分
- **TaskSplitter**：多任务场景拆分

#### 分子专用拆分器
- **ScaffoldSplitter**：按分子骨架结构划分（防止数据泄漏）
- **ButinaSplitter**：基于聚类的分子拆分
- **FingerprintSplitter**：基于分子指纹相似度拆分
- **MaxMinSplitter**：最大化训练集/测试集多样性
- **MolecularWeightSplitter**：按分子量属性拆分

**最佳实践**：药物发现任务中，使用 ScaffoldSplitter 防止相似分子结构过拟合。

### 转换器

#### 标准化
- **NormalizationTransformer**：标准归一化（均值=0，标准差=1）
- **MinMaxTransformer**：特征缩放到 [0,1] 范围
- **LogTransformer**：应用对数转换
- **PowerTransformer**：Box-Cox 和 Yeo-Johnson 转换
- **CDFTransformer**：累积分布函数归一化

#### 任务专用
- **BalancingTransformer**：解决类别不平衡问题
- **FeaturizationTransformer**：应用动态特征工程
- **CoulombFitTransformer**：量子化学专用
- **DAGTransformer**：有向无环图转换
- **RxnSplitTransformer**：化学反应预处理

## 分子特征化器

### 图基特征化器
用于图神经网络（GCNs, MPNNs 等）：
- **ConvMolFeaturizer**：图卷积网络的图表示
- **WeaveFeaturizer**："编织"图嵌入
- **MolGraphConvFeaturizer**：图卷积就绪表示
- **EquivariantGraphFeaturizer**：保持几何不变性
- **DMPNNFeaturizer**：定向消息传递神经网络输入
- **GroverFeaturizer**：预训练分子嵌入

### 指纹基特征化器
用于传统机器学习（随机森林, SVM, XGBoost）：
- **MACCSKeysFingerprint**：167 位结构密钥
- **CircularFingerprint**：扩展连通性指纹（摩根指纹）
  - 参数：`radius`（默认 2），`size`（默认 2048），`useChirality`（默认 False）
- **PubChemFingerprint**：881 位结构描述符
- **Mol2VecFingerprint**：学习型分子向量表示

### 描述符特征化器
直接计算分子属性：
- **RDKitDescriptors**：约 200 种分子描述符（分子量, LogP, 氢供体, 氢受体, TPSA 等）
- **MordredDescriptors**：全面结构及物理化学描述符
- **CoulombMatrix**：用于 3D 结构的原子间距离矩阵

### 序列基特征化器
用于循环网络和 Transformer：
- **SmilesToSeq**：将 SMILES 字符串转为序列
- **SmilesToImage**：从 SMILES 生成 2D 图像表示
- **RawFeaturizer**：原始分子数据直通

### 选择指南

| 使用场景 | 推荐特征化器 | 模型类型 |
|----------|----------------------|------------|
| 图神经网络 | ConvMolFeaturizer, MolGraphConvFeaturizer | GCN, MPNN, GAT |
| 传统机器学习 | CircularFingerprint, RDKitDescriptors | 随机森林, XGBoost, SVM |
| 深度学习（非图） | CircularFingerprint, Mol2VecFingerprint | 密集网络, CNN |
| 序列模型 | SmilesToSeq | LSTM, GRU, Transformer |
| 3D 分子结构 | CoulombMatrix | 专用 3D 模型 |
| 快速基线 | RDKitDescriptors | 线性, Ridge, Lasso |

## 模型

### Scikit-Learn 集成
- **SklearnModel**：任意 scikit-learn 算法的封装器
  - 用法：`SklearnModel(model=RandomForestRegressor())`

### 梯度提升
- **GBDTModel**：梯度提升决策树（XGBoost, LightGBM）

### PyTorch 模型

#### 分子属性预测
- **MultitaskRegressor**：共享表示的多任务回归
- **MultitaskClassifier**：多任务分类
- **MultitaskFitTransformRegressor**：带学习转换的回归
- **GCNModel**：图卷积网络
- **GATModel**：图注意力网络
- **AttentiveFPModel**：注意力指纹网络
- **DMPNNModel**：定向消息传递神经网络
- **GroverModel**：GROVER 预训练 Transformer
- **MATModel**：分子注意力 Transformer

#### 材料科学
- **CGCNNModel**：晶体图卷积网络
- **MEGNetModel**：材料图网络
- **LCNNModel**：材料晶格 CNN

#### 生成模型
- **GANModel**：生成对抗网络
- **WGANModel**：Wasserstein GAN
- **BasicMolGANModel**：分子 GAN
- **LSTMGenerator**：基于 LSTM 的分子生成
- **SeqToSeqModel**：序列到序列模型

#### 物理信息模型
- **PINNModel**：物理信息神经网络
- **HNNModel**：哈密顿神经网络
- **LNN**：拉格朗日神经网络
- **FNOModel**：傅里叶神经算子

#### 计算机视觉
- **CNN**：卷积神经网络
- **UNetModel**：用于分割的 U-Net 架构
- **InceptionV3Model**：预训练 Inception v3
- **MobileNetV2Model**：轻量级移动网络

### Hugging Face 模型

- **HuggingFaceModel**：HF Transformers 通用封装器
- **Chemberta**：用于分子属性预测的化学 BERT
- **MoLFormer**：分子 Transformer 架构
- **ProtBERT**：蛋白质序列 BERT
- **DeepAbLLM**：抗体大语言模型

### 模型选择指南

| 任务 | 推荐模型 | 特征化器 |
|------|------------------|------------|
| 小数据集 (<1000 样本) | SklearnModel (随机森林) | CircularFingerprint |
| 中数据集 (1K-100K) | GBDTModel 或 MultitaskRegressor | CircularFingerprint 或 ConvMolFeaturizer |
| 大数据集 (>100K) | GCNModel, AttentiveFPModel 或 DMPNN | MolGraphConvFeaturizer |
| 迁移学习 | GroverModel, Chemberta, MoLFormer | 模型专用 |
| 材料属性 | CGCNNModel, MEGNetModel | 结构基 |
| 分子生成 | BasicMolGANModel, LSTMGenerator | SmilesToSeq |
| 蛋白质序列 | ProtBERT | 序列基 |

## MoleculeNet 数据集

通过 `dc.molnet.load_*()` 函数快速访问 30+ 基准数据集。

### 分类数据集
- **load_bace()**：BACE-1 抑制剂（二分类）
- **load_bbbp()**：血脑屏障穿透性
- **load_clintox()**：临床毒性
- **load_hiv()**：HIV 抑制活性
- **load_muv()**：PubChem 生物测定（挑战性，稀疏）
- **load_pcba()**：PubChem 筛选数据
- **load_sider()**：药物不良反应（多标签）
- **load_tox21()**：12 项毒性检测（多任务）
- **load_toxcast()**：EPA ToxCast 筛选

### 回归数据集
- **load_delaney()**：水溶性（ESOL）
- **load_freesolv()**：溶剂化自由能
- **load_lipo()**：亲脂性（辛醇-水分配）
- **load_qm7/qm8/qm9()**：量子力学属性
- **load_hopv()**：有机光伏属性

### 蛋白质-配体结合
- **load_pdbbind()**：结合亲和力数据

### 材料科学
- **load_perovskite()**：钙钛矿稳定性
- **load_mp_formation_energy()**：材料项目形成能
- **load_mp_metallicity()**：金属与非金属分类
- **load_bandgap()**：电子带隙预测

### 化学反应
- **load_uspto()**：USPTO 反应数据集

### 使用模式
```python
tasks, datasets, transformers = dc.molnet.load_bbbp(
    featurizer='GraphConv',  # 或 'ECFP', 'GraphConv', 'Weave' 等
    splitter='scaffold',      # 或 'random', 'stratified' 等
    reload=False              # 设为 True 跳过缓存
)
train, valid, test = datasets
```

## 评估指标

`dc.metrics` 提供的常用评估指标：

### 分类指标
- **roc_auc_score**：ROC 曲线下面积（二分类/多分类）
- **prc_auc_score**：精确率-召回率曲线下面积
- **accuracy_score**：分类准确率
- **balanced_accuracy_score**：不平衡数据集的平衡准确率
- **recall_score**：敏感度/召回率
- **precision_score**：精确率
- **f1_score**：F1 分数

### 回归指标
- **mean_absolute_error**：平均绝对误差（MAE）
- **mean_squared_error**：均方误差（MSE）
- **root_mean_squared_error**：均方根误差（RMSE）
- **r2_score**：决定系数 R²
- **pearson_r2_score**：皮尔逊相关系数
- **spearman_correlation**：斯皮尔曼秩相关

### 多任务指标
多数指标支持通过任务平均进行多任务评估。

## 训练模式

标准 DeepChem 工作流：

```python
# 1. 加载数据
loader = dc.data.CSVLoader(tasks=['task1'], feature_field='smiles',
                           featurizer=dc.feat.CircularFingerprint())
dataset = loader.create_dataset('data.csv')

# 2. 拆分数据
splitter = dc.splits.ScaffoldSplitter()
train, valid, test = splitter.train_valid_test_split(dataset)

# 3. 数据转换（可选）
transformers = [dc.trans.NormalizationTransformer(dataset=train)]
for transformer in transformers:
    train = transformer.transform(train)
    valid = transformer.transform(valid)
    test = transformer.transform(test)

# 4. 创建并训练模型
model = dc.models.MultitaskRegressor(n_tasks=1, n_features=2048, layer_sizes=[1000])
model.fit(train, nb_epoch=50)

# 5. 评估
metric = dc.metrics.Metric(dc.metrics.r2_score)
train_score = model.evaluate(train, [metric])
test_score = model.evaluate(test, [metric])
```

## 常用模式

### 模式 1：使用 MoleculeNet 快速基线
```python
tasks, datasets, transformers = dc.molnet.load_tox21(featurizer='ECFP')
train, valid, test = datasets
model = dc.models.MultitaskClassifier(n_tasks=len(tasks), n_features=1024)
model.fit(train)
```

### 模式 2：图网络自定义数据
```python
featurizer = dc.feat.MolGraphConvFeaturizer()
loader = dc.data.CSVLoader(tasks=['activity'], feature_field='smiles',
                           featurizer=featurizer)
dataset = loader.create_dataset('my_data.csv')
train, test = dc.splits.RandomSplitter().train_test_split(dataset)
model = dc.models.GCNModel(mode='classification', n_tasks=1)
model.fit(train)
```

### 模式 3：预训练模型迁移学习
```python
model = dc.models.GroverModel(task='classification', n_tasks=1)
model.fit(train_dataset)
predictions = model.predict(test_dataset)
```
