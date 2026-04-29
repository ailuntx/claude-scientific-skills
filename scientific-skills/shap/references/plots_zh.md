# SHAP 可视化参考指南

本文档全面介绍所有 SHAP 绘图函数及其参数、使用场景和模型解释可视化最佳实践。

## 概述

SHAP 提供多样化可视化工具，用于在个体和全局层面解释模型预测。每种图表类型在理解特征重要性、交互作用和预测机制方面具有特定用途。

## 图表类型

### 瀑布图

**用途**：展示单个预测的解释，呈现每个特征如何将预测值从基线（期望值）推向最终预测结果。

**函数**：`shap.plots.waterfall(explanation, max_display=10, show=True)`

**关键参数**：
- `explanation`：Explanation 对象的单行数据（非多样本）
- `max_display`：显示特征数量（默认：10）；次要特征折叠为"其他特征"项
- `show`：是否立即显示图表

**视觉元素**：
- **X轴**：显示 SHAP 值（对预测的贡献度）
- **起点**：模型期望值（基线）
- **特征贡献**：红色柱（正向）或蓝色柱（负向）展示特征对预测的影响
- **特征值**：灰色显示在特征名称左侧
- **终点**：最终模型预测值

**适用场景**：
- 详细解释单个预测
- 理解驱动特定决策的特征
- 说明单实例模型行为（如贷款拒绝、医疗诊断）
- 调试异常预测

**重要提示**：
- 对于 XGBoost 分类器，预测以对数几率单位解释（逻辑转换前的边际输出）
- SHAP 值总和等于基线与最终预测的差值（可加性特性）
- 结合散点图探索多样本模式

**示例**：
```python
import shap

# 计算 SHAP 值
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)

# 绘制首个预测的瀑布图
shap.plots.waterfall(shap_values[0])

# 显示更多特征
shap.plots.waterfall(shap_values[0], max_display=20)
```

### 蜂群图

**用途**：信息密集型图表，汇总整个数据集中顶级特征对模型输出的影响，结合特征重要性与数值分布。

**函数**：`shap.plots.beeswarm(shap_values, max_display=10, order=Explanation.abs.mean(0), color=None, show=True)`

**关键参数**：
- `shap_values`：包含多样本的 Explanation 对象
- `max_display`：显示特征数量（默认：10）
- `order`：特征排序方式
  - `Explanation.abs.mean(0)`：平均绝对 SHAP 值（默认）
  - `Explanation.abs.max(0)`：最大绝对值（突出异常值影响）
- `color`：matplotlib 配色方案；默认红蓝渐变
- `show`：是否立即显示图表

**视觉元素**：
- **Y轴**：按重要性排序的特征
- **X轴**：SHAP 值（对模型输出的影响）
- **每个点**：数据集中的单个实例
- **点位置（X）**：SHAP 值大小
- **点颜色**：原始特征值（红=高值，蓝=低值）
- **点聚集**：展示影响密度/分布

**适用场景**：
- 总结整个数据集的特征重要性
- 理解特征的平均影响和个体影响
- 识别特征值模式及其效应
- 比较特征的全局模型行为
- 检测非线性关系（如年龄增长→收入可能性降低）

**实用变体**：
```python
# 标准蜂群图
shap.plots.beeswarm(shap_values)

# 显示更多特征
shap.plots.beeswarm(shap_values, max_display=20)

# 按最大绝对值排序（突出异常值）
shap.plots.beeswarm(shap_values, order=shap_values.abs.max(0))

# 固定颜色绘制绝对 SHAP 值
shap.plots.beeswarm(shap_values.abs, color="shap_red")

# 自定义 matplotlib 配色
shap.plots.beeswarm(shap_values, color=plt.cm.viridis)
```

### 条形图

**用途**：以平均绝对 SHAP 值展示特征重要性，提供简洁的全局特征影响可视化。

**函数**：`shap.plots.bar(shap_values, max_display=10, clustering=None, clustering_cutoff=0.5, show=True)`

**关键参数**：
- `shap_values`：Explanation 对象（可接受单实例、全局或分组数据）
- `max_display`：显示的最大特征数/条形数
- `clustering`：来自 `shap.utils.hclust` 的可选层次聚类对象
- `clustering_cutoff`：聚类结构显示阈值（0-1，默认：0.5）

**图表类型**：

#### 全局条形图
展示所有样本的整体特征重要性。重要性按平均绝对 SHAP 值计算。

```python
# 全局特征重要性
explainer = shap.TreeExplainer(model)
shap_values = explainer(X_test)
shap.plots.bar(shap_values)
```

#### 局部条形图
展示单个实例的 SHAP 值，特征值以灰色显示。

```python
# 单预测解释
shap.plots.bar(shap_values[0])
```

#### 分组条形图
通过传递 Explanation 对象字典比较子组特征重要性。

```python
# 比较分组
cohorts = {
    "组 A": shap_values[mask_A],
    "组 B": shap_values[mask_B]
}
shap.plots.bar(cohorts)
```

**特征聚类**：
通过基于模型的聚类识别冗余特征（比基于相关性的方法更准确）。

```python
# 添加特征聚类
clustering = shap.utils.hclust(X_train, y_train)
shap.plots.bar(shap_values, clustering=clustering)

# 调整聚类显示阈值
shap.plots.bar(shap_values, clustering=clustering, clustering_cutoff=0.3)
```

**适用场景**：
- 快速概览全局特征重要性
- 比较不同分组或模型的特征重要性
- 识别冗余或相关特征
- 演示用的简洁可视化

### 力图

**用途**：累加式力图展示特征如何将预测从基线推高（红色）或拉低（蓝色）。

**函数**：`shap.plots.force(base_value, shap_values, features, feature_names=None, out_names=None, link="identity", matplotlib=False, show=True)`

**关键参数**：
- `base_value`：期望值（基线预测）
- `shap_values`：样本的 SHAP 值
- `features`：样本的特征值
- `feature_names`：可选特征名称
- `link`：转换函数（"identity" 或 "logit"）
- `matplotlib`：使用 matplotlib 后端（默认：交互式 JavaScript）

**视觉元素**：
- **基线**：起始预测（期望值）
- **红色箭头**：推高预测的特征
- **蓝色箭头**：拉低预测的特征
- **最终值**：结果预测值

**交互功能**（JavaScript 模式）：
- 悬停查看详细特征信息
- 多样本创建堆叠可视化
- 可旋转切换视角

**适用场景**：
- 交互式预测探索
- 同时可视化多个预测
- 需要交互元素的演示场景
- 快速理解预测构成

**示例**：
```python
# 单预测力图
shap.plots.force(
    shap_values.base_values[0],
    shap_values.values[0],
    X_test.iloc[0],
    matplotlib=True
)

# 多预测交互力图
shap.plots.force(
    shap_values.base_values,
    shap_values.values,
    X_test
)
```

### 散点图（依赖图）

**用途**：展示特征值与其 SHAP 值的关系，揭示特征值如何影响预测。

**函数**：`shap.plots.scatter(shap_values, color=None, hist=True, alpha=1, show=True)`

**关键参数**：
- `shap_values`：Explanation 对象，可通过下标指定特征（如 `shap_values[:, "Age"]`）
- `color`：用于着色的特征（字符串名称或 Explanation 对象）
- `hist`：在 y 轴显示特征值直方图
- `alpha`：点透明度（适用于密集图表）

**视觉元素**：
- **X轴**：特征值
- **Y轴**：SHAP 值（对预测的影响）
- **点颜色**：另一特征的值（用于交互检测）
- **直方图**：特征值分布

**适用场景**：
- 理解特征-预测关系
- 检测非线性效应
- 识别特征交互作用
- 验证或发现模型行为模式
- 探索瀑布图中的反直觉预测

**交互检测**：
通过另一特征着色点以揭示交互作用。

```python
# 基础依赖图
shap.plots.scatter(shap_values[:, "Age"])

# 通过另一特征着色显示交互
shap.plots.scatter(shap_values[:, "Age"], color=shap_values[:, "Education"])

# 单图中多特征展示
shap.plots.scatter(shap_values[:, ["Age", "Education", "Hours-per-week"]])

# 增加密集数据透明度
shap.plots.scatter(shap_values[:, "Age"], alpha=0.5)
```

### 热力图

**用途**：同时可视化多个样本的 SHAP 值，展示跨实例的特征影响。

**函数**：`shap.plots.heatmap(shap_values, instance_order=None, feature_values=None, max_display=10, show=True)`

**关键参数**：
- `shap_values`：Explanation 对象
- `instance_order`：实例排序方式（可用 Explanation 对象自定义排序）
- `feature_values`：悬停时显示特征值
- `max_display`：最大显示特征数

**视觉元素**：
- **行**：单个实例/样本
- **列**：特征
- **单元格颜色**：SHAP 值（红=正向，蓝=负向）
- **颜色强度**：影响大小

**适用场景**：
- 跨实例比较解释
- 识别特征影响模式
- 理解哪些特征在预测间差异最大
- 检测具有相似解释模式的子组或聚类

**示例**：
```python
# 基础热力图
shap.plots.heatmap(shap_values)

# 按模型输出排序实例
shap.plots.heatmap(shap_values, instance_order=shap_values.sum(1))

# 显示特定子集
shap.plots.heatmap(shap_values[:100])
```

### 小提琴图

**用途**：类似蜂群图，但使用小提琴（核密度）可视化替代单点显示。

**函数**：`shap.plots.violin(shap_values, features=None, feature_names=None, max_display=10, show=True)`

**适用场景**：
- 大数据集的蜂群图替代方案
- 强调分布密度而非单点
- 演示用更简洁的可视化

**示例**：
```python
shap.plots.violin(shap_values)
```

### 决策图

**用途**：通过累积 SHAP 值展示预测路径，特别适用于多分类问题。

**函数**：`shap.plots.decision(base_value, shap_values, features, feature_names=None, feature_order="importance", highlight=None, link="identity", show=True)`

**关键参数**：
- `base_value`：期望值
- `shap_values`：样本的 SHAP 值
- `features`：特征值
- `feature_order`：特征排序方式（"importance" 或列表）
- `highlight`：需高亮的样本索引
- `link`：转换函数

**适用场景**：
- 多分类问题解释
- 理解累积特征效应
- 跨样本比较预测路径
- 识别预测分歧点

**示例**：
```python
# 多预测决策图
shap.plots.decision(
    shap_values.base_values,
    shap_values.values,
    X_test,
    feature_names=X_test.columns.tolist()
)

# 高亮特定实例
shap.plots.decision(
    shap_values.base_values,
    shap_values.values,
    X_test,
    highlight=[0, 5, 10]
)
```

## 图表选择指南

**单预测分析**：
- **瀑布图**：最佳详细顺序解释
- **力图**：适合交互探索
- **条形图（局部）**：简洁的单预测重要性展示

**全局理解**：
- **蜂群图**：含数值分布的信息密集型汇总
- **条形图（全局）**：简洁的重要性排序
- **小提琴图**：蜂群图的分布聚焦替代方案

**特征关系分析**：
- **散点图**：理解特征-预测关系及交互
- **热力图**：跨实例比较模式

**多样本分析**：
- **热力图**：SHAP 值网格视图
- **力图（堆叠）**：交互式多样本可视化
- **决策图**：多分类问题的预测路径

**分组比较**：
- **条形图（分组）**：特征重要性清晰对比
- **多蜂群图**：并排分布比较

## 可视化最佳实践

**1. 先全局后局部**：
- 从蜂群图或条形图开始理解全局模式
- 深入瀑布图或散点图分析特定实例/特征

**2. 组合多种图表**：
- 不同图表揭示不同洞察
- 组合瀑布图（个体）+散点图（关系）+蜂群图（全局）

**3. 调整 max_display**：
- 默认值（10）适合演示
- 详细分析可增至（20-30）
- 冗余特征考虑聚类处理

**4. 合理使用颜色**：
- SHAP 值使用默认红蓝配色（红=正向，蓝=负向）
- 散点图通过交互特征着色
- 特定领域使用自定义配色

**5. 考虑受众**：
- 技术受众：蜂群图、散点图、热力图
- 非技术受众：瀑布图、条形图、力图
- 交互演示：JavaScript 版力图

**6. 保存高质量图像**：
```python
import matplotlib.pyplot as plt

# 创建图表
shap.plots.beeswarm(shap_values, show=False)

# 高 DPI 保存
plt.savefig('shap_plot.png', dpi=300, bbox_inches='tight')
plt.close()
```

**7. 处理大数据集**：
- 可视化采样子集（如 `shap_values[:1000]`）
- 超大数据集使用小提琴图替代蜂群图
- 密集散点图调整透明度

## 常见模式与工作流

**模式 1：完整模型解释**
```python
# 1. 全局重要性
shap.plots.beeswarm(shap_values)

# 2. 顶级特征关系
for feature in top_features:
    shap.plots.scatter(shap_values[:, feature])

# 3. 示例预测
for i in interesting_indices:
    shap.plots.waterfall(shap_values[i])
```

**模式 2：模型比较**
```python
# 计算多模型 SHAP 值
shap_model1 = explainer1(X_test)
shap_model2 = explainer2(X_test)

# 比较特征重要性
shap.plots.bar({
    "模型 1": shap_model1,
    "模型 2": shap_model2
})
```

**模式 3：子组分析**
```python
# 定义分组
male_mask = X_test['Sex'] == 'Male'
female_mask = X_test['Sex'] == 'Female'

# 比较分组
shap.plots.bar({
    "男性": shap_values[male_mask],
    "女性": shap_values[female_mask]
})

# 独立蜂群图
shap.plots.beeswarm(shap_values[male_mask])
shap.plots.beeswarm(shap_values[female_mask])
```

**模式 4：预测调试**
```python
# 识别异常或错误
errors = (model.predict(X_test) != y_test)
error_indices = np.where(errors)[0]

# 解释错误
for idx in error_indices[:5]:
    print(f"样本 {idx}:")
    shap.plots.waterfall(shap_values[idx])

    # 探索关键特征
    shap.plots.scatter(shap_values[:, "关键特征"])
```

## 与 Notebook 和报告的集成

**Jupyter Notebook**：
- 交互式力图无缝运行
- 默认 `show=True` 实现内联显示
- 结合 markdown 说明

**静态报告**：
- 力图使用 matplotlib 后端
- 程序化保存图像
- 优先选用瀑布图和条形图确保清晰度

**Web 应用**：
- 将力图导出为 HTML
- 使用 shap.save_html() 生成交互可视化
- 考虑按需生成图表

## 可视化问题排查

**问题**：图表未显示
- **解决方案**：确保正确设置 matplotlib 后端；必要时使用 `plt.show()`

**问题**：过多特征导致图表杂乱
- **解决方案**：减小 `max_display` 参数或使用特征聚类

**问题**：颜色反转或混淆
- **解决方案**：检查模型输出类型（概率 vs. 对数几率）并使用合适的连接函数

**问题**：大型数据集绘图缓慢
- **解决方案**：对数据子集采样；可视化时使用 `shap_values[:1000]`

**问题**：特征名称缺失
- **解决方案**：确保Explanation对象中包含feature_names，或向绘图函数显式传递特征名称
