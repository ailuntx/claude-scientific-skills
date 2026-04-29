---
name: shap
description: 使用SHAP（SHapley Additive exPlanations）实现模型可解释性与可说明性。当需要解释机器学习模型预测、计算特征重要性、生成SHAP图表（瀑布图、蜂群图、条形图、散点图、力图、热力图）、调试模型、分析模型偏见或公平性、比较模型或实施可解释AI时使用此技能。支持树模型（XGBoost、LightGBM、随机森林）、深度学习（TensorFlow、PyTorch）、线性模型及任何黑盒模型。
license: MIT license
metadata:
    skill-author: K-Dense Inc.
---

# SHAP（SHapley可加性解释）

## 概述

SHAP是基于合作博弈论中Shapley值的统一方法，用于解释机器学习模型输出。本技能提供全面指导：

- 为任意模型类型计算SHAP值
- 创建可视化图表理解特征重要性
- 调试与验证模型行为
- 分析公平性与偏见
- 在生产环境中实施可解释AI

SHAP支持所有模型类型：树模型（XGBoost、LightGBM、CatBoost、随机森林）、深度学习模型（TensorFlow、PyTorch、Keras）、线性模型及黑盒模型。

## 使用场景

**当用户询问以下内容时触发此技能**：
- "解释哪些特征对我的模型最重要"
- "生成SHAP图表"（瀑布图、蜂群图、条形图、散点图、力图、热力图等）
- "为什么我的模型做出这个预测？"
- "为我的模型计算SHAP值"
- "使用SHAP可视化特征重要性"
- "调试模型行为"或"验证模型"
- "检查模型偏见"或"分析公平性"
- "跨模型比较特征重要性"
- "实施可解释AI"或"为模型添加解释"
- "理解特征交互作用"
- "创建模型解释仪表板"

## 快速入门指南

### 步骤1：选择合适解释器

**决策树**：

1. **树模型？**（XGBoost、LightGBM、CatBoost、随机森林、梯度提升）
   - 使用`shap.TreeExplainer`（快速、精确）

2. **深度神经网络？**（TensorFlow、PyTorch、Keras、CNN、RNN、Transformer）
   - 使用`shap.DeepExplainer`或`shap.GradientExplainer`

3. **线性模型？**（线性/逻辑回归、GLM）
   - 使用`shap.LinearExplainer`（极速）

4. **其他模型？**（SVM、自定义函数、黑盒模型）
   - 使用`shap.KernelExplainer`（模型无关但较慢）

5. **不确定？**
   - 使用`shap.Explainer`（自动选择最佳算法）

**详见`references/explainers.md`获取所有解释器类型的详细信息。**

### 步骤2：计算SHAP值

```python
import shap

# 树模型示例（XGBoost）
import xgboost as xgb

# 训练模型
model = xgb.XGBClassifier().fit(X_train, y_train)

# 创建解释器
explainer = shap.TreeExplainer(model)

# 计算SHAP值
shap_values = explainer(X_test)

# shap_values对象包含：
# - values: SHAP值（特征贡献度）
# - base_values: 模型预期输出（基线）
# - data: 原始特征值
```

### 步骤3：可视化结果

**全局理解**（完整数据集）：
```python
# 蜂群图 - 展示特征重要性及数值分布
shap.plots.beeswarm(shap_values, max_display=15)

# 条形图 - 特征重要性简明摘要
shap.plots.bar(shap_values)
```

**单样本预测解释**：
```python
# 瀑布图 - 单次预测的详细分解
shap.plots.waterfall(shap_values[0])

# 力图 - 加性作用力可视化
shap.plots.force(shap_values[0])
```

**特征关联分析**：
```python
# 散点图 - 特征与预测值关系
shap.plots.scatter(shap_values[:, "特征名称"])

# 通过另一特征着色展示交互作用
shap.plots.scatter(shap_values[:, "年龄"], color=shap_values[:, "教育程度"])
```

**详见`references/plots.md`获取所有图表类型的完整指南。**

## 核心工作流

本技能支持多种常见工作流，请根据当前任务选择。

### 工作流1：基础模型解释

**目标**：理解模型预测驱动因素

**步骤**：
1. 训练模型并创建对应解释器
2. 为测试集计算SHAP值
3. 生成全局重要性图表（蜂群图/条形图）
4. 分析关键特征关联（散点图）
5. 解释特定预测（瀑布图）

**示例**：
```python
# 步骤1-2：初始化
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)

# 步骤3：全局重要性
shap.plots.beeswarm(shap_values)

# 步骤4：特征关联
shap.plots.scatter(shap_values[:, "最重要特征"])

# 步骤5：单样本解释
shap.plots.waterfall(shap_values[0])
```

### 工作流2：模型调试

**目标**：识别并修复模型问题

**步骤**：
1. 计算SHAP值
2. 定位预测错误
3. 解释误分类样本
4. 检查异常特征重要性（数据泄露）
5. 验证特征关联合理性
6. 检查特征交互作用

**详见`references/workflows.md`获取详细调试流程。**

### 工作流3：特征工程

**目标**：利用SHAP洞察优化特征

**步骤**：
1. 为基准模型计算SHAP值
2. 识别非线性关系（需转换的候选特征）
3. 识别特征交互（需创建交互项的候选）
4. 构建新特征
5. 重新训练并对比SHAP值
6. 验证改进效果

**详见`references/workflows.md`获取详细特征工程流程。**

### 工作流4：模型比较

**目标**：对比多个模型选择最佳可解释方案

**步骤**：
1. 训练多个模型
2. 分别计算SHAP值
3. 比较全局特征重要性
4. 检查特征排序一致性
5. 分析跨模型特定预测
6. 基于准确性、可解释性和一致性选择

**详见`references/workflows.md`获取详细模型比较流程。**

### 工作流5：公平性与偏见分析

**目标**：检测并分析跨人口群体的模型偏见

**步骤**：
1. 识别受保护属性（性别、种族、年龄等）
2. 计算SHAP值
3. 对比不同群体的特征重要性
4. 检查受保护属性的SHAP重要性
5. 识别代理特征
6. 发现偏见时实施缓解策略

**详见`references/workflows.md`获取详细公平性分析流程。**

### 工作流6：生产部署

**目标**：将SHAP解释集成至生产系统

**步骤**：
1. 训练并保存模型
2. 创建并保存解释器
3. 构建解释服务
4. 创建带解释的预测API端点
5. 实施缓存与优化
6. 监控解释质量

**详见`references/workflows.md`获取详细生产部署流程。**

## 核心概念

### SHAP值

**定义**：SHAP值量化每个特征对预测的贡献度，以偏离模型预期输出（基线）的程度衡量。

**特性**：
- **可加性**：SHAP值总和等于预测值与基线的差值
- **公平性**：基于博弈论中的Shapley值
- **一致性**：特征重要性增加时其SHAP值随之增大

**解读**：
- 正值 → 特征推高预测结果
- 负值 → 特征拉低预测结果
- 绝对值 → 特征影响强度
- SHAP值总和 → 预测值相对基线的总变化

**示例**：
```
基线（预期值）：0.30
特征贡献（SHAP值）：
  年龄：+0.15
  收入：+0.10
  教育程度：-0.05
最终预测：0.30 + 0.15 + 0.10 - 0.05 = 0.50
```

### 背景数据/基线

**作用**：代表"典型"输入以建立基准预期

**选择方法**：
- 训练数据随机抽样（50-1000样本）
- 或使用kmeans选取代表性样本
- DeepExplainer/KernelExplainer：100-1000样本平衡精度与速度

**影响**：基线影响SHAP值量级但不改变相对重要性

### 模型输出类型

**关键考量**：明确模型输出形式

- **原始输出**：回归或树模型边际值
- **概率值**：分类概率
- **对数几率**：逻辑回归（Sigmoid函数前）

**示例**：XGBoost分类器默认解释边际输出（对数几率）。要解释概率，需在TreeExplainer中设置`model_output="probability"`。

## 常用模式

### 模式1：完整模型分析

```python
# 1. 初始化
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)

# 2. 全局重要性
shap.plots.beeswarm(shap_values)
shap.plots.bar(shap_values)

# 3. 关键特征关联
top_features = X_test.columns[np.abs(shap_values.values).mean(0).argsort()[-5:]]
for feature in top_features:
    shap.plots.scatter(shap_values[:, feature])

# 4. 样本预测解释
for i in range(5):
    shap.plots.waterfall(shap_values[i])
```

### 模式2：群体对比

```python
# 定义群体
cohort1_mask = X_test['分组'] == 'A'
cohort2_mask = X_test['分组'] == 'B'

# 对比特征重要性
shap.plots.bar({
    "A组": shap_values[cohort1_mask],
    "B组": shap_values[cohort2_mask]
})
```

### 模式3：错误调试

```python
# 定位错误样本
errors = model.predict(X_test) != y_test
error_indices = np.where(errors)[0]

# 解释错误
for idx in error_indices[:5]:
    print(f"样本 {idx}:")
    shap.plots.waterfall(shap_values[idx])

    # 调查关键特征
    shap.plots.scatter(shap_values[:, "可疑特征"])
```

## 性能优化

### 速度考量

**解释器速度**（从快到慢）：
1. `LinearExplainer` - 近实时
2. `TreeExplainer` - 极快
3. `DeepExplainer` - 神经网络场景快速
4. `GradientExplainer` - 神经网络场景快速
5. `KernelExplainer` - 较慢（必要时使用）
6. `PermutationExplainer` - 极慢但精确

### 优化策略

**大型数据集**：
```python
# 计算子集SHAP值
shap_values = explainer(X_test[:1000])

# 或使用分批处理
batch_size = 100
all_shap_values = []
for i in range(0, len(X_test), batch_size):
    batch_shap = explainer(X_test[i:i+batch_size])
    all_shap_values.append(batch_shap)
```

**可视化优化**：
```python
# 采样子集绘图
shap.plots.beeswarm(shap_values[:1000])

# 调整密集图透明度
shap.plots.scatter(shap_values[:, "特征"], alpha=0.3)
```

**生产环境**：
```python
# 缓存解释器
import joblib
joblib.dump(explainer, 'explainer.pkl')
explainer = joblib.load('explainer.pkl')

# 为批量预测预计算
# API响应仅计算前N个特征
```

## 故障排除

### 问题：解释器选择错误
**现象**：树模型使用KernelExplainer（低效且非必需）
**解决**：树模型始终使用TreeExplainer

### 问题：背景数据不足
**现象**：DeepExplainer/KernelExplainer背景样本过少
**解决**：使用100-1000个代表性样本

### 问题：单位混淆
**现象**：将对数几率误认为概率
**解决**：检查模型输出类型，明确数值是概率/对数几率/原始输出

### 问题：图表无法显示
**现象**：Matplotlib后端问题
**解决**：确保正确设置后端，必要时使用`plt.show()`

### 问题：特征过多导致图表混乱
**现象**：默认max_display=10可能不适用
**解决**：调整`max_display`参数或使用特征聚类

### 问题：计算缓慢
**现象**：超大数据集计算SHAP
**解决**：采样子集、分批处理或确保使用专用解释器（非KernelExplainer）

## 工具集成

### Jupyter Notebooks
- 交互式力图无缝支持
- 默认`show=True`内联显示图表
- 结合Markdown构建叙述性解释

### MLflow/实验跟踪
```python
import mlflow

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

    # 记录特征重要性指标
    mean_abs_shap = np.abs(shap_values.values).mean(axis=0)
    for feature, importance in zip(X_test.columns, mean_abs_shap):
        mlflow.log_metric(f"shap_{feature}", importance)
```

### 生产API
```python
class ExplanationService:
    def __init__(self, model_path, explainer_path):
        self.model = joblib.load(model_path)
        self.explainer = joblib.load(explainer_path)

    def predict_with_explanation(self, X):
        prediction = self.model.predict(X)
        shap_values = self.explainer(X)

        return {
            'prediction': prediction[0],
            'base_value': shap_values.base_values[0],
            'feature_contributions': dict(zip(X.columns, shap_values.values[0]))
        }
```

## 参考文档

本技能包含按主题组织的完整参考文档：

### references/explainers.md
所有解释器类完整指南：
- `TreeExplainer` - 树模型快速精确解释
- `DeepExplainer` - 深度学习模型（TensorFlow、PyTorch）
- `KernelExplainer` - 模型无关（支持任意模型）
- `LinearExplainer` - 线性模型快速解释
- `GradientExplainer` - 神经网络梯度解释
- `PermutationExplainer` - 精确但速度较慢

包含：构造函数参数、方法、支持模型、适用场景、示例、性能考量。

### references/plots.md
完整可视化指南：
- **瀑布图** - 单样本预测分解
- **蜂群图** - 全局重要性及数值分布
- **条形图** - 特征重要性摘要
- **散点图** - 特征-预测关系及交互
- **力图** - 交互式加性作用力
- **热力图** - 多样本对比矩阵
- **小提琴图** - 分布导向替代方案
- **决策图** - 多类别预测路径

包含：参数配置、用例场景、示例、最佳实践、图表选择指南。

### references/workflows.md
详细工作流与最佳实践：
- 基础模型解释流程
- 模型调试与验证
- 特征工程指导
- 模型比较与选择
- 公平性与偏见分析
- 深度学习模型解释
- 生产环境部署
- 时间序列模型解释
- 常见陷阱与解决方案
- 高级技术
- MLOps集成

包含：分步指导、代码示例、决策标准、故障排除。

### references/theory.md
理论基础：
- 博弈论中的Shapley值
- 数学公式与特性
- 与其他解释方法的关联（LIME、DeepLIFT等）
- SHAP计算算法（Tree SHAP、Kernel SHAP等）
- 条件期望与基线选择
- SHAP值解读
-

- 当用户询问理论基础、Shapley值或数学细节时，加载 `theory.md`

**默认方法**（不加载参考文件）：
- 使用本SKILL.md进行基础解释和快速入门
- 提供标准工作流程和常见模式
- 如需更多细节可查阅参考文件

**加载参考文件**：
```python
# 加载参考文件时，使用Read工具并指定正确路径：
# /path/to/shap/references/explainers.md
# /path/to/shap/references/plots.md
# /path/to/shap/references/workflows.md
# /path/to/shap/references/theory.md
```

## 最佳实践摘要

1. **选择合适解释器**：尽可能使用专用解释器（TreeExplainer, DeepExplainer, LinearExplainer）；除非必要避免使用KernelExplainer

2. **先全局后局部**：从蜂群图/条形图开始建立整体认知，再通过瀑布图/散点图深入细节

3. **组合多种可视化**：不同图表揭示不同洞察；结合全局（蜂群图）+局部（瀑布图）+关系（散点图）视图

4. **选择合适背景数据**：使用训练数据中50-1000个代表性样本

5. **理解模型输出单位**：明确解释的是概率、对数几率还是原始输出

6. **用领域知识验证**：SHAP展示模型行为；需结合领域知识进行解释和验证

7. **性能优化**：可视化时采样子集，大数据集分批处理，生产环境缓存解释器

8. **检查数据泄露**：意外的高特征重要性可能暗示数据质量问题

9. **考虑特征相关性**：对冗余特征使用TreeExplainer的相关性感知选项或特征聚类

10. **谨记SHAP显示关联而非因果**：因果解释需依赖领域知识

## 安装指南

```bash
# 基础安装
uv pip install shap

# 含可视化依赖
uv pip install shap matplotlib

# 最新版本
uv pip install -U shap
```

**依赖项**：numpy, pandas, scikit-learn, matplotlib, scipy

**可选依赖**：xgboost, lightgbm, tensorflow, torch（取决于模型类型）

## 扩展资源

- **官方文档**：https://shap.readthedocs.io/
- **GitHub仓库**：https://github.com/slundberg/shap
- **原始论文**：Lundberg & Lee (2017) - "解释模型预测的统一方法"
- **Nature MI论文**：Lundberg et al. (2020) - "从局部解释到全局理解：基于树模型的可解释AI"

本技能全面覆盖SHAP在各种使用场景和模型类型中的可解释性应用。
