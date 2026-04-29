# PyHealth 训练、评估与可解释性

## 概述

PyHealth 提供全面的工具链，用于模型训练、预测评估、模型可靠性保障以及临床应用的模型结果解释。

## Trainer 类

### 核心功能

`Trainer` 类通过 PyTorch 集成管理完整的模型训练与评估工作流。

**初始化：**
```python
from pyhealth.trainer import Trainer

trainer = Trainer(
    model=model,  # PyHealth 或 PyTorch 模型
    device="cuda",  # 或 "cpu"
)
```

### 训练

**train() 方法**

提供全面监控与检查点保存的模型训练功能。

**参数：**
- `train_dataloader`: 训练数据加载器
- `val_dataloader`: 验证数据加载器（可选）
- `test_dataloader`: 测试数据加载器（可选）
- `epochs`: 训练轮次
- `optimizer`: 优化器实例或类
- `learning_rate`: 学习率（默认：1e-3）
- `weight_decay`: L2 正则化（默认：0）
- `max_grad_norm`: 梯度裁剪阈值
- `monitor`: 监控指标（如 "pr_auc_score"）
- `monitor_criterion`: "max" 或 "min"
- `save_path`: 检查点保存目录

**用法：**
```python
trainer.train(
    train_dataloader=train_loader,
    val_dataloader=val_loader,
    test_dataloader=test_loader,
    epochs=50,
    optimizer=torch.optim.Adam,
    learning_rate=1e-3,
    weight_decay=1e-5,
    max_grad_norm=5.0,
    monitor="pr_auc_score",
    monitor_criterion="max",
    save_path="./checkpoints"
)
```

**训练特性：**

1. **自动检查点保存**：根据监控指标保存最佳模型
2. **早停机制**：性能无提升时停止训练
3. **梯度裁剪**：防止梯度爆炸
4. **进度跟踪**：实时显示训练进度与指标
5. **多GPU支持**：自动设备分配

### 推理

**inference() 方法**

在数据集上执行预测。

**参数：**
- `dataloader`: 推理数据加载器
- `additional_outputs`: 需返回的附加输出列表
- `return_patient_ids`: 是否返回患者标识符

**用法：**
```python
predictions = trainer.inference(
    dataloader=test_loader,
    additional_outputs=["attention_weights", "embeddings"],
    return_patient_ids=True
)
```

**返回：**
- `y_pred`: 模型预测结果
- `y_true`: 真实标签
- `patient_ids`: 患者标识符（如请求）
- 附加输出（如指定）

### 评估

**evaluate() 方法**

计算综合评估指标。

**参数：**
- `dataloader`: 评估数据加载器
- `metrics`: 指标函数列表

**用法：**
```python
from pyhealth.metrics import binary_metrics_fn

results = trainer.evaluate(
    dataloader=test_loader,
    metrics=["accuracy", "pr_auc_score", "roc_auc_score", "f1_score"]
)

print(results)
# 输出: {'accuracy': 0.85, 'pr_auc_score': 0.78, 'roc_auc_score': 0.82, 'f1_score': 0.73}
```

### 检查点管理

**save() 方法**
```python
trainer.save("./models/best_model.pt")
```

**load() 方法**
```python
trainer.load("./models/best_model.pt")
```

## 评估指标

### 二分类指标

**可用指标：**
- `accuracy`: 整体准确率
- `precision`: 查准率
- `recall`: 召回率/敏感度
- `f1_score`: F1分数（精确率与召回率调和平均）
- `roc_auc_score`: ROC曲线下面积
- `pr_auc_score`: 精确率-召回率曲线下面积
- `cohen_kappa`: 评分者间一致性

**用法：**
```python
from pyhealth.metrics import binary_metrics_fn

# 综合二分类指标
metrics = binary_metrics_fn(
    y_true=labels,
    y_pred=predictions,
    metrics=["accuracy", "f1_score", "pr_auc_score", "roc_auc_score"]
)
```

**阈值选择：**
```python
# 默认阈值: 0.5
predictions_binary = (predictions > 0.5).astype(int)

# 基于F1的最优阈值
from sklearn.metrics import f1_score
thresholds = np.arange(0.1, 0.9, 0.05)
f1_scores = [f1_score(y_true, (y_pred > t).astype(int)) for t in thresholds]
optimal_threshold = thresholds[np.argmax(f1_scores)]
```

**最佳实践：**
- **使用AUROC**：评估模型整体区分能力
- **使用AUPRC**：尤其适用于不平衡类别
- **使用F1**：平衡精确率与召回率
- **报告置信区间**：通过自助采样法计算

### 多分类指标

**可用指标：**
- `accuracy`: 整体准确率
- `macro_f1`: 各类别F1的未加权均值
- `micro_f1`: 全局F1（基于总TP/FP/FN）
- `weighted_f1`: 按类别频率加权的F1均值
- `cohen_kappa`: 多分类Kappa系数

**用法：**
```python
from pyhealth.metrics import multiclass_metrics_fn

metrics = multiclass_metrics_fn(
    y_true=labels,
    y_pred=predictions,
    metrics=["accuracy", "macro_f1", "weighted_f1"]
)
```

**分类别指标：**
```python
from sklearn.metrics import classification_report

print(classification_report(y_true, y_pred,
    target_names=["清醒", "N1", "N2", "N3", "REM"]))
```

**混淆矩阵：**
```python
from sklearn.metrics import confusion_matrix
import seaborn as sns

cm = confusion_matrix(y_true, y_pred)
sns.heatmap(cm, annot=True, fmt='d')
```

### 多标签分类指标

**可用指标：**
- `jaccard_score`: 交并比
- `hamming_loss`: 错误标签比例
- `example_f1`: 样本级F1（微平均）
- `label_f1`: 标签级F1（宏平均）

**用法：**
```python
from pyhealth.metrics import multilabel_metrics_fn

# y_pred: [n_samples, n_labels] 二元矩阵
metrics = multilabel_metrics_fn(
    y_true=label_matrix,
    y_pred=pred_matrix,
    metrics=["jaccard_score", "example_f1", "label_f1"]
)
```

**药物推荐指标：**
```python
# Jaccard相似度（交集/并集）
jaccard = len(set(true_drugs) & set(pred_drugs)) / len(set(true_drugs) | set(pred_drugs))

# Precision@k: 前k个预测的精确率
def precision_at_k(y_true, y_pred, k=10):
    top_k_pred = y_pred.argsort()[-k:]
    return len(set(y_true) & set(top_k_pred)) / k
```

### 回归指标

**可用指标：**
- `mean_absolute_error`: 平均绝对误差
- `mean_squared_error`: 均方误差
- `root_mean_squared_error`: 均方根误差
- `r2_score`: 决定系数

**用法：**
```python
from pyhealth.metrics import regression_metrics_fn

metrics = regression_metrics_fn(
    y_true=true_values,
    y_pred=predictions,
    metrics=["mae", "rmse", "r2"]
)
```

**百分比误差指标：**
```python
# 平均绝对百分比误差
mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100

# 中位数绝对百分比误差（抗异常值）
medape = np.median(np.abs((y_true - y_pred) / y_true)) * 100
```

### 公平性指标

**目的：** 评估模型在不同人口统计群体中的偏差

**可用指标：**
- `demographic_parity`: 群体间正例预测率相等
- `equalized_odds`: 群体间TPR与FPR相等
- `equal_opportunity`: 群体间TPR相等
- `predictive_parity`: 群体间PPV相等

**用法：**
```python
from pyhealth.metrics import fairness_metrics_fn

fairness_results = fairness_metrics_fn(
    y_true=labels,
    y_pred=predictions,
    sensitive_attributes=demographics,  # 如种族、性别
    metrics=["demographic_parity", "equalized_odds"]
)
```

**示例：**
```python
# 评估性别公平性
male_mask = (demographics == "male")
female_mask = (demographics == "female")

male_tpr = recall_score(y_true[male_mask], y_pred[male_mask])
female_tpr = recall_score(y_true[female_mask], y_pred[female_mask])

tpr_disparity = abs(male_tpr - female_tpr)
print(f"TPR差异: {tpr_disparity:.3f}")
```

## 校准与不确定性量化

### 模型校准

**目的：** 确保预测概率与实际频率匹配

**校准曲线：**
```python
from sklearn.calibration import calibration_curve
import matplotlib.pyplot as plt

fraction_of_positives, mean_predicted_value = calibration_curve(
    y_true, y_prob, n_bins=10
)

plt.plot(mean_predicted_value, fraction_of_positives, marker='o')
plt.plot([0, 1], [0, 1], linestyle='--', label='完美校准')
plt.xlabel('预测概率均值')
plt.ylabel('正例比例')
plt.legend()
```

**期望校准误差 (ECE)：**
```python
def expected_calibration_error(y_true, y_prob, n_bins=10):
    """计算 ECE"""
    bins = np.linspace(0, 1, n_bins + 1)
    bin_indices = np.digitize(y_prob, bins) - 1

    ece = 0
    for i in range(n_bins):
        mask = bin_indices == i
        if mask.sum() > 0:
            bin_accuracy = y_true[mask].mean()
            bin_confidence = y_prob[mask].mean()
            ece += mask.sum() / len(y_true) * abs(bin_accuracy - bin_confidence)

    return ece
```

**校准方法：**

1. **Platt缩放**：在验证预测上应用逻辑回归
```python
from sklearn.linear_model import LogisticRegression

calibrator = LogisticRegression()
calibrator.fit(val_predictions.reshape(-1, 1), val_labels)
calibrated_probs = calibrator.predict_proba(test_predictions.reshape(-1, 1))[:, 1]
```

2. **保序回归**：非参数校准方法
```python
from sklearn.isotonic import IsotonicRegression

calibrator = IsotonicRegression(out_of_bounds='clip')
calibrator.fit(val_predictions, val_labels)
calibrated_probs = calibrator.predict(test_predictions)
```

3. **温度缩放**：在softmax前缩放logits
```python
def find_temperature(logits, labels):
    """寻找最优温度参数"""
    from scipy.optimize import minimize

    def nll(temp):
        scaled_logits = logits / temp
        probs = torch.softmax(scaled_logits, dim=1)
        return F.cross_entropy(probs, labels).item()

    result = minimize(nll, x0=1.0, method='BFGS')
    return result.x[0]

temperature = find_temperature(val_logits, val_labels)
calibrated_logits = test_logits / temperature
```

### 不确定性量化

**保形预测：**

提供具有统计保证覆盖率的预测集。

**用法：**
```python
from pyhealth.metrics import prediction_set_metrics_fn

# 在验证集上校准
scores = 1 - val_predictions[np.arange(len(val_labels)), val_labels]
quantile_level = np.quantile(scores, 0.9)  # 90% 覆盖率

# 在测试集生成预测集
prediction_sets = test_predictions > (1 - quantile_level)

# 评估
metrics = prediction_set_metrics_fn(
    y_true=test_labels,
    prediction_sets=prediction_sets,
    metrics=["coverage", "average_size"]
)
```

**蒙特卡洛Dropout：**

通过推理时dropout估计不确定性。

```python
def predict_with_uncertainty(model, dataloader, num_samples=20):
    """使用MC dropout进行不确定性预测"""
    model.train()  # 保持dropout激活

    predictions = []
    for _ in range(num_samples):
        batch_preds = []
        for batch in dataloader:
            with torch.no_grad():
                output = model(batch)
                batch_preds.append(output)
        predictions.append(torch.cat(batch_preds))

    predictions = torch.stack(predictions)
    mean_pred = predictions.mean(dim=0)
    std_pred = predictions.std(dim=0)  # 不确定性

    return mean_pred, std_pred
```

**集成不确定性：**

```python
# 训练多个模型
models = [train_model(seed=i) for i in range(5)]

# 集成预测
ensemble_preds = []
for model in models:
    pred = model.predict(test_data)
    ensemble_preds.append(pred)

mean_pred = np.mean(ensemble_preds, axis=0)
std_pred = np.std(ensemble_preds, axis=0)  # 不确定性
```

## 可解释性

### 注意力可视化

**适用于Transformer和RETAIN模型：**

```python
# 推理时获取注意力权重
outputs = trainer.inference(
    test_loader,
    additional_outputs=["attention_weights"]
)

attention = outputs["attention_weights"]

# 可视化样本注意力
import matplotlib.pyplot as plt
import seaborn as sns

sample_idx = 0
sample_attention = attention[sample_idx]  # [seq_length, seq_length]

sns.heatmap(sample_attention, cmap='viridis')
plt.xlabel('键位置')
plt.ylabel('查询位置')
plt.title('注意力权重')
plt.show()
```

**RETAIN解释：**

```python
# RETAIN提供就诊级和特征级注意力
visit_attention = outputs["visit_attention"]  # 重要就诊
feature_attention = outputs["feature_attention"]  # 重要特征

# 找出最具影响力的就诊
most_important_visit = visit_attention[sample_idx].argmax()

# 找出该就诊中最重要的特征
important_features = feature_attention[sample_idx, most_important_visit].argsort()[-10:]
```

### 特征重要性

**置换重要性：**

```python
from sklearn.inspection import permutation_importance

def get_predictions(model, X):
    return model.predict(X)

result = permutation_importance(
    model, X_test, y_test,
    n_repeats=10,
    scoring='roc_auc'
)

# 按重要性排序特征
indices = result.importances_mean.argsort()[::-1]
for i in indices[:10]:
    print(f"{feature_names[i]}: {result.importances_mean[i]:.3f}")
```

**SHAP值：**

```python
import shap

# 创建解释器
explainer = shap.DeepExplainer(model, train_data)

# 计算SHAP值
shap_values = explainer.shap_values(test_data)

# 可视化
shap.summary_plot(shap_values, test_data, feature_names=feature_names)
```

### ChEFER (临床健康事件特征提取与排序)

**PyHealth的可解释性工具：**

```python
from pyhealth.explain import ChEFER

explainer = ChEFER(model=model, dataset=test_dataset)

# 获取预测的特征重要性
importance_scores = explainer.explain(
    patient_id="patient_123",
    visit_id="visit_456"
)

# 可视化重要特征
explainer.plot_importance(importance_scores, top_k=20)
```

## 完整训练流程示例

```python
from pyhealth.datasets import MIMIC4Dataset
from pyhealth.tasks import mortality_prediction_mimic4_fn
from pyhealth.datasets import split_by_patient, get_dataloader
from pyhealth.models import Transformer
from pyhealth.trainer import Trainer
from pyhealth.metrics import binary_metrics_fn

# 1. 加载与准备数据
dataset = MIMIC4Dataset(root="/path/to/mimic4")
sample_dataset = dataset.set_task(mortality_prediction_mimic4_fn

# 5. 训练模型
trainer = Trainer(model=model, device="cuda")
trainer.train(
    train_dataloader=train_loader,
    val_dataloader=val_loader,
    epochs=50,
    optimizer=torch.optim.Adam,
    learning_rate=1e-3,
    weight_decay=1e-5,
    monitor="pr_auc_score",
    monitor_criterion="max",
    save_path="./checkpoints/mortality_model"
)

# 6. 在测试集上评估
test_results = trainer.evaluate(
    test_loader,
    metrics=["accuracy", "precision", "recall", "f1_score",
             "roc_auc_score", "pr_auc_score"]
)

print("测试结果：")
for metric, value in test_results.items():
    print(f"{metric}: {value:.4f}")

# 7. 获取预测结果用于分析
predictions = trainer.inference(test_loader, return_patient_ids=True)
y_pred, y_true, patient_ids = predictions

# 8. 校准分析
from sklearn.calibration import calibration_curve

fraction_pos, mean_pred = calibration_curve(y_true, y_pred, n_bins=10)
ece = expected_calibration_error(y_true, y_pred)
print(f"预期校准误差: {ece:.4f}")

# 9. 保存最终模型
trainer.save("./models/mortality_transformer_final.pt")
```

## 最佳实践

### 训练

1. **监控多个指标**：同时跟踪损失值和任务特定指标
2. **使用验证集**：通过早停防止过拟合
3. **梯度裁剪**：稳定训练过程 (max_grad_norm=5.0)
4. **学习率调度**：在平台期降低学习率
5. **保存最佳模型检查点**：基于验证集表现保存

### 评估

1. **使用任务适配指标**：二分类用AUROC/AUPRC，不平衡多分类用宏平均F1
2. **报告置信区间**：使用自助法或交叉验证
3. **分层评估**：按亚组报告指标
4. **临床相关指标**：包含临床决策阈值
5. **公平性评估**：跨人口统计学组别评估

### 部署

1. **校准预测结果**：确保概率可靠性
2. **量化不确定性**：提供置信度估计
3. **监控生产环境表现**：持续跟踪指标
4. **处理分布偏移**：检测数据变化
5. **可解释性**：提供预测解释
