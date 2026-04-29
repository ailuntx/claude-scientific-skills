# SHAP 工作流程与最佳实践

本文档提供了在各种模型解释场景中使用 SHAP 的完整工作流程、最佳实践和常见用例。

## 基础工作流程结构

所有 SHAP 分析都遵循通用工作流程：

1. **训练模型**：构建并训练机器学习模型
2. **选择解释器**：根据模型类型选择合适的解释器
3. **计算 SHAP 值**：为测试样本生成解释
4. **可视化结果**：使用图表理解特征影响
5. **解释与行动**：得出结论并制定决策

## 工作流程 1：基础模型解释

**用例**：理解已训练模型的特征重要性和预测行为

```python
import shap
import pandas as pd
from sklearn.model_selection import train_test_split

# 步骤 1：加载并分割数据
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 步骤 2：训练模型（XGBoost 示例）
import xgboost as xgb
model = xgb.XGBClassifier(n_estimators=100, max_depth=5)
model.fit(X_train, y_train)

# 步骤 3：创建解释器
explainer = shap.TreeExplainer(model)

# 步骤 4：计算 SHAP 值
shap_values = explainer(X_test)

# 步骤 5：可视化全局重要性
shap.plots.beeswarm(shap_values, max_display=15)

# 步骤 6：详细检查关键特征
shap.plots.scatter(shap_values[:, "Feature1"])
shap.plots.scatter(shap_values[:, "Feature2"], color=shap_values[:, "Feature1"])

# 步骤 7：解释单样本预测
shap.plots.waterfall(shap_values[0])
```

**关键决策**：
- 基于模型架构选择解释器类型
- 背景数据集大小（针对 DeepExplainer, KernelExplainer）
- 需解释的样本数量（全测试集 vs 子集）

## 工作流程 2：模型调试与验证

**用例**：识别并修复模型问题，验证预期行为

```python
# 步骤 1：计算 SHAP 值
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)

# 步骤 2：识别预测错误
predictions = model.predict(X_test)
errors = predictions != y_test
error_indices = np.where(errors)[0]

# 步骤 3：分析错误样本
print(f"总错误数: {len(error_indices)}")
print(f"错误率: {len(error_indices) / len(y_test):.2%}")

# 步骤 4：解释误分类样本
for idx in error_indices[:10]:  # 前10个错误
    print(f"\n=== 错误 {idx} ===")
    print(f"预测值: {predictions[idx]}, 实际值: {y_test.iloc[idx]}")
    shap.plots.waterfall(shap_values[idx])

# 步骤 5：检查模型是否学习正确模式
# 寻找异常特征重要性
shap.plots.beeswarm(shap_values)

# 步骤 6：研究特定特征关系
# 验证非线性关系合理性
for feature in model.feature_importances_.argsort()[-5:]:  # 前5重要特征
    feature_name = X_test.columns[feature]
    shap.plots.scatter(shap_values[:, feature_name])

# 步骤 7：验证特征交互
# 检查交互是否符合领域知识
shap.plots.scatter(shap_values[:, "Feature1"], color=shap_values[:, "Feature2"])
```

**需检查的常见问题**：
- 数据泄露（重要性异常高的特征）
- 伪相关性（意外的特征关系）
- 目标泄露（不应具有预测性的特征）
- 偏见（对特定群体的不均衡影响）

## 工作流程 3：特征工程指导

**用例**：利用 SHAP 洞察改进特征工程

```python
# 步骤 1：使用基线特征训练初始模型
model_v1 = train_model(X_train_v1, y_train)
explainer_v1 = shap.TreeExplainer(model_v1)
shap_values_v1 = explainer_v1(X_test_v1)

# 步骤 2：识别特征工程机会
shap.plots.beeswarm(shap_values_v1)

# 检查以下情况：
# - 非线性关系（转换候选特征）
shap.plots.scatter(shap_values_v1[:, "Age"])  # 可能需要 age^2 或分箱？

# - 特征交互（交互项候选）
shap.plots.scatter(shap_values_v1[:, "Income"], color=shap_values_v1[:, "Education"])
# 可创建 Income * Education 交互项？

# 步骤 3：基于洞察构建新特征
X_train_v2 = X_train_v1.copy()
X_train_v2['Age_squared'] = X_train_v2['Age'] ** 2
X_train_v2['Income_Education'] = X_train_v2['Income'] * X_train_v2['Education']

# 步骤 4：使用新特征重新训练
model_v2 = train_model(X_train_v2, y_train)
explainer_v2 = shap.TreeExplainer(model_v2)
shap_values_v2 = explainer_v2(X_test_v2)

# 步骤 5：比较特征重要性
shap.plots.bar({
    "基线模型": shap_values_v1,
    "含新特征模型": shap_values_v2
})

# 步骤 6：验证改进效果
print(f"V1 分数: {model_v1.score(X_test_v1, y_test):.4f}")
print(f"V2 分数: {model_v2.score(X_test_v2, y_test):.4f}")
```

**SHAP 的特征工程洞察**：
- 强非线性模式 → 尝试转换（log, sqrt, 多项式）
- 散点图中的颜色编码交互 → 创建交互项
- 聚类中的冗余特征 → 删除或合并
- 意外重要性 → 调查数据质量问题

## 工作流程 4：模型比较与选择

**用例**：比较多个模型以选择最佳可解释模型

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
import xgboost as xgb

# 步骤 1：训练多个模型
models = {
    '逻辑回归': LogisticRegression(max_iter=1000).fit(X_train, y_train),
    '随机森林': RandomForestClassifier(n_estimators=100).fit(X_train, y_train),
    'XGBoost': xgb.XGBClassifier(n_estimators=100).fit(X_train, y_train)
}

# 步骤 2：为每个模型计算 SHAP 值
shap_values_dict = {}
for name, model in models.items():
    if name == '逻辑回归':
        explainer = shap.LinearExplainer(model, X_train)
    else:
        explainer = shap.TreeExplainer(model)
    shap_values_dict[name] = explainer(X_test)

# 步骤 3：比较全局特征重要性
shap.plots.bar(shap_values_dict)

# 步骤 4：比较模型分数
for name, model in models.items():
    score = model.score(X_test, y_test)
    print(f"{name}: {score:.4f}")

# 步骤 5：检查特征重要性一致性
for feature in X_test.columns[:5]:  # 前5个特征
    fig, axes = plt.subplots(1, 3, figsize=(15, 4))
    for idx, (name, shap_vals) in enumerate(shap_values_dict.items()):
        plt.sca(axes[idx])
        shap.plots.scatter(shap_vals[:, feature], show=False)
        plt.title(f"{name} - {feature}")
    plt.tight_layout()
    plt.show()

# 步骤 6：跨模型分析特定预测
sample_idx = 0
for name, shap_vals in shap_values_dict.items():
    print(f"\n=== {name} ===")
    shap.plots.waterfall(shap_vals[sample_idx])

# 步骤 7：基于以下因素决策：
# - 准确率/性能
# - 可解释性（一致的特征重要性）
# - 部署限制
# - 利益相关方需求
```

**模型选择标准**：
- **准确率 vs 可解释性**：有时带 SHAP 的简单模型更可取
- **特征一致性**：特征重要性一致的模型更可信
- **解释质量**：清晰、可操作的说明
- **计算成本**：TreeExplainer 比 KernelExplainer 更快

## 工作流程 5：公平性与偏见分析

**用例**：检测并分析跨人口群体的模型偏见

```python
# 步骤 1：识别受保护属性
protected_attr = '性别'  # 或 '种族', '年龄组' 等

# 步骤 2：计算 SHAP 值
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)

# 步骤 3：跨群体比较特征重要性
groups = X_test[protected_attr].unique()
cohorts = {
    f"{protected_attr}={group}": shap_values[X_test[protected_attr] == group]
    for group in groups
}
shap.plots.bar(cohorts)

# 步骤 4：检查受保护属性的 SHAP 重要性
# （公平模型中应接近零）
protected_importance = np.abs(shap_values[:, protected_attr].values).mean()
print(f"{protected_attr} 平均 |SHAP|: {protected_importance:.4f}")

# 步骤 5：分析各群体预测
for group in groups:
    mask = X_test[protected_attr] == group
    group_shap = shap_values[mask]

    print(f"\n=== {protected_attr} = {group} ===")
    print(f"样本量: {mask.sum()}")
    print(f"正向预测率: {(model.predict(X_test[mask]) == 1).mean():.2%}")

    # 可视化
    shap.plots.beeswarm(group_shap, max_display=10)

# 步骤 6：检查代理特征
# 与受保护属性相关但不应高重要性的特征
# 例如：'邮编' 可能是种族的代理
proxy_features = ['邮编', '姓氏前缀']  # 领域特定
for feature in proxy_features:
    if feature in X_test.columns:
        importance = np.abs(shap_values[:, feature].values).mean()
        print(f"潜在代理 '{feature}' 重要性: {importance:.4f}")

# 步骤 7：发现偏见后的缓解策略
# - 移除受保护属性及代理
# - 训练时添加公平性约束
# - 后处理预测以平衡结果
# - 使用不同模型架构
```

**需检查的公平性指标**：
- **人口均等**：跨群体相似的正向预测率
- **机会均等**：跨群体相似的真正例率
- **特征重要性均等**：跨群体相似的特征排序
- **受保护属性重要性**：应最小化

## 工作流程 6：深度学习模型解释

**用例**：使用 DeepExplainer 解释神经网络预测

```python
import tensorflow as tf
import shap

# 步骤 1：加载或构建神经网络
model = tf.keras.models.load_model('my_model.h5')

# 步骤 2：选择背景数据集
# 使用训练数据子集（100-1000样本）
background = X_train[:100]

# 步骤 3：创建 DeepExplainer
explainer = shap.DeepExplainer(model, background)

# 步骤 4：计算 SHAP 值（可能耗时）
# 解释测试数据子集
test_subset = X_test[:50]
shap_values = explainer.shap_values(test_subset)

# 步骤 5：处理多输出模型
# 二分类中 shap_values 为列表 [类0值, 类1值]
# 回归中为单数组
if isinstance(shap_values, list):
    # 聚焦正类
    shap_values_positive = shap_values[1]
    shap_exp = shap.Explanation(
        values=shap_values_positive,
        base_values=explainer.expected_value[1],
        data=test_subset
    )
else:
    shap_exp = shap.Explanation(
        values=shap_values,
        base_values=explainer.expected_value,
        data=test_subset
    )

# 步骤 6：可视化
shap.plots.beeswarm(shap_exp)
shap.plots.waterfall(shap_exp[0])

# 步骤 7：图像/文本数据使用专用图表
# 图像：shap.image_plot
# 文本：shap.plots.text（针对 transformer）
```

**深度学习注意事项**：
- 背景数据集大小影响精度和速度
- 多输出处理（分类 vs 回归）
- 图像/文本数据的专用图表
- 计算成本（考虑 GPU 加速）

## 工作流程 7：生产部署

**用例**：将 SHAP 解释集成到生产系统

```python
import joblib
import shap

# 步骤 1：训练并保存模型
model = train_model(X_train, y_train)
joblib.dump(model, 'model.pkl')

# 步骤 2：创建并保存解释器
explainer = shap.TreeExplainer(model)
joblib.dump(explainer, 'explainer.pkl')

# 步骤 3：创建解释服务
class ExplanationService:
    def __init__(self, model_path, explainer_path):
        self.model = joblib.load(model_path)
        self.explainer = joblib.load(explainer_path)

    def predict_with_explanation(self, X):
        """
        返回预测结果及解释
        """
        # 预测
        prediction = self.model.predict(X)

        # SHAP 值
        shap_values = self.explainer(X)

        # 格式化解释
        explanations = []
        for i in range(len(X)):
            exp = {
                'prediction': prediction[i],
                'base_value': shap_values.base_values[i],
                'shap_values': dict(zip(X.columns, shap_values.values[i])),
                'feature_values': X.iloc[i].to_dict()
            }
            explanations.append(exp)

        return explanations

    def get_top_features(self, X, n=5):
        """
        返回每个预测的前 N 个特征
        """
        shap_values = self.explainer(X)

        top_features = []
        for i in range(len(X)):
            # 获取绝对 SHAP 值
            abs_shap = np.abs(shap_values.values[i])

            # 排序并取前 N
            top_indices = abs_shap.argsort()[-n:][::-1]
            top_feature_names = X.columns[top_indices].tolist()
            top_shap_values = shap_values.values[i][top_indices].tolist()

            top_features.append({
                'features': top_feature_names,
                'shap_values': top_shap_values
            })

        return top_features

# 步骤 4：在 API 中使用
service = ExplanationService('model.pkl', 'explainer.pkl')

# API 端点示例
def predict_endpoint(input_data):
    X = pd.DataFrame([input_data])
    explanations = service.predict_with_explanation(X)
    return {
        'prediction': explanations[0]['prediction'],
        'explanation': explanations[0]
    }

# 步骤 5：为批量预测生成静态解释
def batch_explain_and_save(X_batch, output_dir):
    shap_values = explainer(X_batch)

    # 保存全局图表
    shap.plots.beeswarm(shap_values, show=False)
    plt.savefig(f'{output_dir}/global_importance.png', dpi=300, bbox_inches='tight')
    plt.close()

    # 保存单样本解释
    for i in range(min(100, len(X_batch))):  # 前100个
        shap.plots.waterfall(shap_values[i], show=False)
        plt.savefig(f'{output_dir}/explanation_{i}.png', dpi=300, bbox_inches='tight')
        plt.close()
```

**生产环境最佳实践**：
- 缓存解释器避免重复计算
- 尽可能批量生成解释
- 限制解释复杂度（仅显示前 N 特征）
- 监控解释延迟
- 解释器与模型同步版本管理
- 为常见输入预计算解释

## 工作流程 8：时间序列模型解释

**用例**：解释时间序列预测模型

```python
# 步骤 1：准备含时间特征的数据
# 示例：预测次日销售额
df['DayOfWeek'] = df['Date'].dt.dayofweek
df['Month'] = df['Date'].dt.month
df['Lag_1'] = df['Sales'].shift(1)
df['Lag_7'] = df['Sales'].shift(7)
df['Rolling_Mean_7'] = df['Sales'].rolling(7).mean()

# 步骤 2：训练模型
features = ['DayOfWeek', 'Month', 'Lag_1', 'Lag_7', 'Rolling_Mean_7']
X_train, X_test, y_train, y_test = train_test_split(df[features], df['Sales'])
model = xgb.XGBRegressor().fit(X_train, y_train)

# 步骤 3：计算 SH

lag_features = ['Lag_1', 'Lag_7', 'Rolling_Mean_7']
for feature in lag_features:
    shap.plots.scatter(shap_values[:, feature])

# 步骤6：解释特定预测
# 例如：为何周一的预测差异如此显著？
monday_mask = X_test['DayOfWeek'] == 0
shap.plots.waterfall(shap_values[monday_mask][0])

# 步骤7：验证季节性理解
shap.plots.scatter(shap_values[:, 'Month'])
```

**时间序列注意事项**：
- 滞后特征及其重要性
- 滚动统计量的解读
- SHAP值中的季节性模式
- 特征工程中避免数据泄露

## 常见陷阱与解决方案

### 陷阱1：解释器选择错误
**问题**：对树模型使用KernelExplainer（低效且不必要）
**解决方案**：树模型始终使用TreeExplainer

### 陷阱2：背景数据不足
**问题**：DeepExplainer/KernelExplainer背景样本过少
**解决方案**：使用100-1000个代表性样本

### 陷阱3：对数几率误解
**问题**：单位混淆（概率 vs 对数几率）
**解决方案**：检查模型输出类型；必要时使用link="logit"

### 陷阱4：忽略特征相关性
**问题**：将相关特征误判为独立变量
**解决方案**：使用特征聚类；理解领域关联性

### 陷阱5：过度依赖解释结果
**问题**：仅基于SHAP进行特征工程而未验证
**解决方案**：始终通过交叉验证确认改进效果

### 陷阱6：未检测数据泄露
**问题**：忽略异常特征重要性暗示的数据泄露
**解决方案**：对照领域知识验证SHAP结果

### 陷阱7：忽视计算限制
**问题**：为整个大型数据集计算SHAP值
**解决方案**：采用抽样、分批或子集分析

## 高级技巧

### 技巧1：SHAP交互值
捕获特征间交互效应：
```python
explainer = shap.TreeExplainer(model)
shap_interaction_values = explainer.shap_interaction_values(X_test)

# 分析特定交互
feature1_idx = 0
feature2_idx = 3
interaction = shap_interaction_values[:, feature1_idx, feature2_idx]
print(f"交互强度: {np.abs(interaction).mean():.4f}")
```

### 技巧2：结合偏依赖图
将偏依赖图与SHAP结合：
```python
from sklearn.inspection import partial_dependence

# SHAP依赖图
shap.plots.scatter(shap_values[:, "Feature1"])

# 偏依赖图（模型无关）
pd_result = partial_dependence(model, X_test, features=["Feature1"])
plt.plot(pd_result['grid_values'][0], pd_result['average'][0])
```

### 技巧3：条件期望分析
基于其他特征条件分析SHAP值：
```python
# 高收入组
high_income = X_test['Income'] > X_test['Income'].median()
shap.plots.beeswarm(shap_values[high_income])

# 低收入组
low_income = X_test['Income'] <= X_test['Income'].median()
shap.plots.beeswarm(shap_values[low_income])
```

### 技巧4：冗余特征聚类
```python
# 创建层次聚类
clustering = shap.utils.hclust(X_train, y_train)

# 带聚类的可视化
shap.plots.bar(shap_values, clustering=clustering, clustering_cutoff=0.5)

# 识别待移除冗余特征
# 距离<0.1的特征高度冗余
```

## MLOps集成

**实验追踪**：
```python
import mlflow

# 记录SHAP值
with mlflow.start_run():
    # 训练模型
    model = train_model(X_train, y_train)

    # 计算SHAP
    explainer = shap.TreeExplainer(model)
    shap_values = explainer(X_test)

    # 记录图表
    shap.plots.beeswarm(shap_values, show=False)
    mlflow.log_figure(plt.gcf(), "shap_beeswarm.png")
    plt.close()

    # 将特征重要性记录为指标
    mean_abs_shap = np.abs(shap_values.values).mean(axis=0)
    for feature, importance in zip(X_test.columns, mean_abs_shap):
        mlflow.log_metric(f"shap_{feature}", importance)
```

**模型监控**：
```python
# 追踪SHAP分布随时间漂移
def compute_shap_summary(shap_values):
    return {
        'mean': shap_values.values.mean(axis=0),
        'std': shap_values.values.std(axis=0),
        'percentiles': np.percentile(shap_values.values, [25, 50, 75], axis=0)
    }

# 计算基线
baseline_summary = compute_shap_summary(shap_values_train)

# 监控生产数据
production_summary = compute_shap_summary(shap_values_production)

# 检测漂移
drift_detected = np.abs(
    production_summary['mean'] - baseline_summary['mean']
) > threshold
```

本文档涵盖了SHAP实践中最常见和高级的应用场景。
