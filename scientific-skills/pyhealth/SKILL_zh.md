```markdown
---
name: pyhealth
description: 面向临床数据的综合性医疗人工智能工具包，用于开发、测试和部署机器学习模型。该技能适用于处理电子健康记录（EHR）、临床预测任务（死亡率、再入院、用药推荐）、医疗编码系统（ICD、NDC、ATC）、生理信号（EEG、ECG）、医疗数据集（MIMIC-III/IV、eICU、OMOP）或实现医疗深度学习模型（RETAIN、SafeDrug、Transformer、GNN）。
license: MIT 许可证
metadata:
    skill-author: K-Dense Inc.
---

# PyHealth：医疗人工智能工具包

## 概述

PyHealth 是一个面向医疗人工智能的综合性 Python 库，提供临床机器学习所需的专用工具、模型和数据集。在开发医疗预测模型、处理临床数据、使用医疗编码系统或在医疗环境中部署人工智能解决方案时使用此技能。

## 使用场景

在以下场景调用此技能：

- **处理医疗数据集**：MIMIC-III、MIMIC-IV、eICU、OMOP、睡眠脑电图数据、医学影像
- **临床预测任务**：死亡率预测、再入院率预测、住院时长预测、用药推荐
- **医疗编码**：ICD-9/10、NDC、RxNorm、ATC 编码系统间的转换
- **处理临床数据**：时序事件、生理信号、临床文本、医学影像
- **实现医疗模型**：RETAIN、SafeDrug、GAMENet、StageNet、Transformer for EHR
- **评估临床模型**：公平性指标、校准、可解释性、不确定性量化

## 核心能力

PyHealth 通过模块化的五阶段医疗 AI 流水线运作：

1. **数据加载**：通过标准化接口访问 10+ 医疗数据集
2. **任务定义**：应用 20+ 预定义临床预测任务或创建自定义任务
3. **模型选择**：从 33+ 模型中选择（基线模型、深度学习模型、医疗专用模型）
4. **训练**：支持自动检查点保存、监控和评估的训练流程
5. **部署**：为临床使用进行校准、解释和验证

**性能**：医疗数据处理速度比 pandas 快 3 倍

## 快速入门

```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.datasets import split_by_patient, get_dataloader
from pyhealth.models import Transformer
from pyhealth.trainer import Trainer

# 1. 加载数据集并设置任务
dataset = MIMIC4Dataset(root="/path/to/data")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)

# 2. 分割数据
train, val, test = split_by_patient(sample_dataset, [0.7, 0.1, 0.2])

# 3. 创建数据加载器
train_loader = get_dataloader(train, batch_size=64, shuffle=True)
val_loader = get_dataloader(val, batch_size=64, shuffle=False)
test_loader = get_dataloader(test, batch_size=64, shuffle=False)

# 4. 初始化并训练模型
model = Transformer(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "medications"],
    mode="binary",
    embedding_dim=128
)

trainer = Trainer(model=model, device="cuda")
trainer.train(
    train_dataloader=train_loader,
    val_dataloader=val_loader,
    epochs=50,
    monitor="pr_auc_score"
)

# 5. 评估模型
results = trainer.evaluate(test_loader)
```

## 详细文档

本技能包含按功能组织的完整参考文档，按需查阅：

### 1. 数据集与数据结构

**文件**：`references/datasets.md`

**阅读场景**：
- 加载医疗数据集（MIMIC、eICU、OMOP、睡眠脑电图等）
- 理解事件、患者、就诊数据结构
- 处理不同数据类型（EHR、信号、影像、文本）
- 分割训练/验证/测试数据
- 使用 SampleDataset 进行任务特定格式化

**核心主题**：
- 核心数据结构（事件、患者、就诊）
- 10+ 可用数据集（EHR、生理信号、影像、文本）
- 数据加载与迭代
- 训练/验证/测试分割策略
- 大型数据集性能优化

### 2. 医疗编码转换

**文件**：`references/medical_coding.md`

**阅读场景**：
- 医疗编码系统间转换
- 处理诊断编码（ICD-9-CM、ICD-10-CM、CCS）
- 处理药物编码（NDC、RxNorm、ATC）
- 标准化操作编码（ICD-9-PROC、ICD-10-PROC）
- 将编码分组为临床类别
- 处理分层药物分类

**核心主题**：
- 系统内查询的 InnerMap
- 跨系统转换的 CrossMap
- 支持的编码系统（ICD、NDC、ATC、CCS、RxNorm）
- 编码标准化与层级遍历
- 按治疗类别进行药物分类
- 与数据集集成

### 3. 临床预测任务

**文件**：`references/tasks.md`

**阅读场景**：
- 定义临床预测目标
- 使用预定义任务（死亡率、再入院率、用药推荐）
- 处理基于 EHR、信号、影像或文本的任务
- 创建自定义预测任务
- 为模型设置输入/输出模式
- 应用任务特定过滤逻辑

**核心主题**：
- 20+ 预定义临床任务
- EHR 任务（死亡率、再入院率、住院时长、用药推荐）
- 信号任务（睡眠分期、脑电图分析、癫痫检测）
- 影像任务（COVID-19 胸部 X 光分类）
- 文本任务（医疗编码、专科分类）
- 自定义任务创建模式

### 4. 模型与架构

**文件**：`references/models.md`

**阅读场景**：
- 选择临床预测模型
- 理解模型架构与能力
- 在通用模型与医疗专用模型间选择
- 实现可解释模型（RETAIN、AdaCare）
- 处理用药推荐（SafeDrug、GAMENet）
- 使用图神经网络处理医疗数据
- 配置模型超参数

**核心主题**：
- 33+ 可用模型
- 通用模型：逻辑回归、MLP、CNN、RNN、Transformer、GNN
- 医疗专用模型：RETAIN、SafeDrug、GAMENet、StageNet、AdaCare
- 按任务类型和数据类型的模型选择
- 可解释性考量
- 计算资源需求
- 超参数调优指南

### 5. 数据预处理

**文件**：`references/preprocessing.md`

**阅读场景**：
- 为模型预处理临床数据
- 处理时序事件与时间序列数据
- 处理生理信号（EEG、ECG）
- 标准化检验值与生命体征
- 为不同任务类型准备标签
- 构建特征词汇表
- 管理缺失数据与异常值

**核心主题**：
- 15+ 处理器类型
- 序列处理（填充、截断）
- 信号处理（滤波、分段）
- 特征提取与编码
- 标签处理器（二分类、多分类、多标签、回归）
- 文本与影像预处理
- 常见预处理流程

### 6. 训练与评估

**文件**：`references/training_evaluation.md`

**阅读场景**：
- 使用 Trainer 类训练模型
- 评估模型性能
- 计算临床指标
- 评估跨人口统计组的模型公平性
- 校准预测可靠性
- 量化预测不确定性
- 解释模型预测
- 准备临床部署模型

**核心主题**：
- Trainer 类（训练、评估、推理）
- 二分类/多分类/多标签/回归任务指标
- 偏差评估的公平性指标
- 校准方法（Platt 缩放、温度缩放）
- 不确定性量化（保形预测、MC dropout）
- 可解释性工具（注意力可视化、SHAP、ChEFER）
- 完整训练流程示例

## 安装

```bash
uv pip install pyhealth
```

**要求**：
- Python ≥ 3.7
- PyTorch ≥ 1.8
- NumPy、pandas、scikit-learn

## 典型用例

### 用例 1：ICU 死亡率预测

**目标**：预测重症监护病房患者死亡率

**步骤**：
1. 加载 MIMIC-IV 数据集 → 阅读 `references/datasets.md`
2. 应用死亡率预测任务 → 阅读 `references/tasks.md`
3. 选择可解释模型（RETAIN） → 阅读 `references/models.md`
4. 训练与评估 → 阅读 `references/training_evaluation.md`
5. 为临床使用解释预测 → 阅读 `references/training_evaluation.md`

### 用例 2：安全用药推荐

**目标**：推荐药物同时避免药物相互作用

**步骤**：
1. 加载 EHR 数据集（MIMIC-IV 或 OMOP） → 阅读 `references/datasets.md`
2. 应用用药推荐任务 → 阅读 `references/tasks.md`
3. 使用带 DDI 约束的 SafeDrug 模型 → 阅读 `references/models.md`
4. 预处理药物编码 → 阅读 `references/medical_coding.md`
5. 使用多标签指标评估 → 阅读 `references/training_evaluation.md`

### 用例 3：再入院率预测

**目标**：识别 30 天再入院高风险患者

**步骤**：
1. 加载多中心 EHR 数据（eICU 或 OMOP） → 阅读 `references/datasets.md`
2. 应用再入院预测任务 → 阅读 `references/tasks.md`
3. 预处理中处理类别不平衡 → 阅读 `references/preprocessing.md`
4. 训练 Transformer 模型 → 阅读 `references/models.md`
5. 校准预测并评估公平性 → 阅读 `references/training_evaluation.md`

### 用例 4：睡眠障碍诊断

**目标**：根据脑电信号分类睡眠阶段

**步骤**：
1. 加载睡眠脑电数据集（SleepEDF、SHHS） → 阅读 `references/datasets.md`
2. 应用睡眠分期任务 → 阅读 `references/tasks.md`
3. 预处理脑电信号（滤波、分段） → 阅读 `references/preprocessing.md`
4. 训练 CNN 或 RNN 模型 → 阅读 `references/models.md`
5. 评估分阶段性能 → 阅读 `references/training_evaluation.md`

### 用例 5：医疗编码转换

**目标**：在不同编码系统间标准化诊断

**步骤**：
1. 阅读 `references/medical_coding.md` 获取完整指南
2. 使用 CrossMap 在 ICD-9、ICD-10、CCS 间转换
3. 将编码分组为临床意义类别
4. 与数据集处理集成

### 用例 6：临床文本转 ICD 编码

**目标**：从临床笔记自动分配 ICD 编码

**步骤**：
1. 加载含临床文本的 MIMIC-III → 阅读 `references/datasets.md`
2. 应用 ICD 编码任务 → 阅读 `references/tasks.md`
3. 预处理临床文本 → 阅读 `references/preprocessing.md`
4. 使用 TransformersModel（ClinicalBERT） → 阅读 `references/models.md`
5. 使用多标签指标评估 → 阅读 `references/training_evaluation.md`

## 最佳实践

### 数据处理

1. **始终按患者分割**：确保患者不跨分割组以防止数据泄露
   ```python
   from pyhealth.datasets import split_by_patient
   train, val, test = split_by_patient(dataset, [0.7, 0.1, 0.2])
   ```

2. **检查数据集统计**：建模前理解数据分布
   ```python
   print(dataset.stats())  # 患者数、就诊数、事件数、编码分布
   ```

3. **使用合适预处理**：根据数据类型匹配处理器（见 `references/preprocessing.md`）

### 模型开发

1. **从基线开始**：用简单模型建立基准性能
   - 二分类/多分类任务用逻辑回归
   - 深度学习基线用 MLP

2. **选择任务适配模型**：
   - 需可解释性 → RETAIN、AdaCare
   - 用药推荐 → SafeDrug、GAMENet
   - 长序列 → Transformer
   - 图关系 → GNN

3. **监控验证指标**：根据任务选用指标并处理类别不平衡
   - 二分类：AUROC、AUPRC（尤其稀有事件）
   - 多分类：macro-F1（不平衡数据）、weighted-F1
   - 多标签：Jaccard、example-F1
   - 回归：MAE、RMSE

### 临床部署

1. **校准预测**：确保概率可靠性（见 `references/training_evaluation.md`）

2. **评估公平性**：跨人口统计组检测偏差

3. **量化不确定性**：提供预测置信度估计

4. **解释预测**：使用注意力权重、SHAP 或 ChEFER 建立临床信任

5. **全面验证**：使用不同时段或中心的保留测试集

## 限制与考量

### 数据要求

- **大型数据集**：深度学习模型需充足数据（数千患者）
- **数据质量**：缺失数据和编码错误影响性能
- **时序一致性**：需要时确保训练/测试分割遵循时间顺序

### 临床验证

- **外部验证**：在不同医院/系统数据上测试
- **前瞻性评估**：部署前在真实临床环境验证
- **临床审查**：由临床医生审核预测和解释
- **伦理考量**：解决隐私（HIPAA/GDPR）、公平性和安全性

### 计算资源

- **推荐 GPU**：高效训练深度学习模型
- **内存要求**：大型数据集需 16GB+ RAM
- **存储空间**：医疗数据集可能达 10-100GB 级

## 故障排除

### 常见问题

**数据集导入错误**：
- 确认数据集文件已下载且路径正确
- 检查 PyHealth 版本兼容性

**内存不足**：
- 减小批处理大小
- 缩短序列长度（`max_seq_length`）
- 使用梯度累积
- 分块处理数据

**性能不佳**：
- 检查类别不平衡并使用合适指标（AUPRC vs AUROC）
- 验证预处理（标准化、缺失数据处理）
- 增加模型容量或训练轮次
- 检查训练/测试分割是否存在数据泄露

**训练缓慢**：
- 使用 GPU（`device="cuda"`）
- 增大批处理大小（若内存允许）
- 缩短序列长度
- 使用更高效模型（CNN 替代 Transformer）

### 获取帮助

- **文档**：https://pyhealth.readthedocs.io/
- **GitHub Issues**：https://github.com/sunlabuiuc/PyHealth/issues
- **教程**：在线提供 7 个核心教程 + 5 个实践流程

## 完整流程示例

```python
# 完整死亡率预测流程
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.datasets import split_by_patient, get_dataloader
from pyhealth.models import RETAIN
from pyhealth.trainer import Trainer

# 1. 加载数据集
print("加载 MIMIC-IV 数据集...")
dataset = MIMIC4Dataset(root="/data/mimic4")
print(dataset.stats())

# 2. 定义任务
print("设置死亡率预测任务...")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn)
print(f"生成 {len(sample_dataset)} 个样本")

# 3. 分割数据（按患者防止泄露）
print("分割数据...")
train_ds, val_ds, test_ds = split_by_patient(
    sample_dataset, ratios=[0.7, 0.1, 0.2], seed=42
)

# 4. 创建数据加载器
train_loader = get_dataloader(train_ds, batch_size=64, shuffle=True)
val_loader = get_dataloader(val_ds, batch_size=64)
test_loader = get_dataloader(test_ds, batch_size=64)

# 5. 初始化可解释模型
print("初始化 RETAIN 模型...")
model = RETAIN(
    dataset=sample_dataset,
    feature_keys=["diagnoses", "procedures", "medications"],
    mode="binary",
    embedding_dim=128,
    hidden_dim=128
)

# 6. 训练模型
print("训练模型...")
trainer = Trainer(model=model, device="cuda")
trainer.train(
    train_dataloader=train_loader,
    val_dataloader=

high_risk_idx = predictions["y_pred"].argmax()
patient_id = predictions["patient_ids"][high_risk_idx]
visit_attn = predictions["visit_attention"][high_risk_idx]
feature_attn = predictions["feature_attention"][high_risk_idx]

print(f"\n高风险患者：{patient_id}")
print(f"风险评分：{predictions['y_pred'][high_risk_idx]:.3f}")
print(f"最具影响力的就诊记录：{visit_attn.argmax()}")
print(f"最重要的特征：{feature_attn[visit_attn.argmax()].argsort()[-5:]}")

# 10. 保存模型以供部署
trainer.save("./models/mortality_retain_final.pt")
print("\n模型保存成功！")
```

## 资源

有关各组件的详细信息，请查阅 `references/` 目录中的完整参考文件：

- **datasets.md**：数据结构、加载与拆分（4,500 词）
- **medical_coding.md**：代码转换与标准化（3,800 词）
- **tasks.md**：临床预测任务及自定义任务创建（4,200 词）
- **models.md**：模型架构与选择指南（5,100 词）
- **preprocessing.md**：数据处理器与预处理流程（4,600 词）
- **training_evaluation.md**：训练、指标、校准与可解释性（5,900 词）

**总计完整文档**：模块化参考文件共约 28,000 词。
