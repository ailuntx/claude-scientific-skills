# SHAP 解释器参考指南

本文档提供所有 SHAP 解释器类的完整信息，包括参数说明、方法详解及适用场景。

## 概述

SHAP 为不同模型类型提供专用解释器，每种解释器针对特定架构进行优化。通用类 `shap.Explainer` 会根据模型类型自动选择最合适的算法。

## 核心解释器类

### shap.Explainer（自动选择器）

**功能**：通过选择最合适的解释算法，自动使用 Shapley 值解释任何机器学习模型或 Python 函数。

**构造参数**：
- `model`：待解释模型（函数或模型对象）
- `masker`：用于特征操作的背景数据或掩码对象
- `algorithm`：可选参数，强制指定解释器类型
- `output_names`：模型输出名称
- `feature_names`：输入特征名称

**适用场景**：不确定解释器选择时的默认方案；根据模型类型自动选择最优算法。

### TreeExplainer

**功能**：基于 Tree SHAP 算法，为树集成模型提供快速精确的 SHAP 值计算。

**构造参数**：
- `model`：树模型（XGBoost、LightGBM、CatBoost、PySpark 或 scikit-learn 树模型）
- `data`：特征整合背景数据集（tree_path_dependent 模式下可选）
- `feature_perturbation`：特征依赖处理方式
  - `"interventional"`：需背景数据；遵循因果推断规则
  - `"tree_path_dependent"`：无需背景数据；使用叶节点训练样本
  - `"auto"`：提供数据时默认 interventional，否则使用 tree_path_dependent
- `model_output`：待解释的模型输出
  - `"raw"`：标准模型输出（默认）
  - `"probability"`：概率转换输出
  - `"log_loss"`：损失函数的自然对数
  - 自定义方法名如 `"predict_proba"`
- `feature_names`：可选特征命名

**支持模型**：
- XGBoost (xgboost.XGBClassifier, xgboost.XGBRegressor, xgboost.Booster)
- LightGBM (lightgbm.LGBMClassifier, lightgbm.LGBMRegressor, lightgbm.Booster)
- CatBoost (catboost.CatBoostClassifier, catboost.CatBoostRegressor)
- PySpark MLlib 树模型
- scikit-learn (DecisionTreeClassifier, DecisionTreeRegressor, RandomForestClassifier, RandomForestRegressor, ExtraTreesClassifier, ExtraTreesRegressor, GradientBoostingClassifier, GradientBoostingRegressor)

**核心方法**：
- `shap_values(X)`：计算样本 SHAP 值；返回每行代表特征归因的数组
- `shap_interaction_values(X)`：估算特征间交互效应；提供主效应和成对交互矩阵
- `explain_row(row)`：解释单行数据并输出详细归因信息

**适用场景**：
- 所有树模型的首选方案
- 需要精确 SHAP 值（非近似值）时
- 处理大型数据集需保证计算速度时
- 适用于随机森林、梯度提升或 XGBoost 等模型

**示例**：
```python
import shap
import xgboost

# 训练模型
model = xgboost.XGBClassifier().fit(X_train, y_train)

# 创建解释器
explainer = shap.TreeExplainer(model)

# 计算 SHAP 值
shap_values = explainer.shap_values(X_test)

# 计算交互值
shap_interaction = explainer.shap_interaction_values(X_test)
```

### DeepExplainer

**功能**：使用改进版 DeepLIFT 算法为深度学习模型近似计算 SHAP 值。

**构造参数**：
- `model`：框架相关定义
  - **TensorFlow**：(输入张量, 输出张量) 元组（输出需单维）
  - **PyTorch**：`nn.Module` 对象或 `(模型, 层)` 元组（支持分层解释）
- `data`：特征整合背景数据集
  - **TensorFlow**：numpy 数组或 pandas DataFrame
  - **PyTorch**：torch 张量
  - **推荐规模**：100-1000 样本（非全量训练集），平衡精度与计算成本
- `session`（仅限 TensorFlow）：可选会话对象；空值时自动检测
- `learning_phase_flags`：自定义学习阶段张量，用于推理时处理批归一化/丢弃层

**支持框架**：
- **TensorFlow**：完整支持（含 Keras 模型）
- **PyTorch**：全面兼容 nn.Module 架构

**核心方法**：
- `shap_values(X)`：返回模型应用于数据 X 的近似 SHAP 值
- `explain_row(row)`：解释单行数据并输出归因值与期望输出
- `save(file)` / `load(file)`：解释器对象序列化支持
- `supports_model_with_masker(model, masker)`：模型类型兼容性检查器

**适用场景**：
- 解释 TensorFlow 或 PyTorch 的深度神经网络
- 处理卷积神经网络（CNN）
- 适用于循环神经网络（RNN）和 Transformer
- 需要针对深度学习架构进行模型专属解释时

**关键设计特性**：
期望方差估计值按 1/√N 比例缩放（N 为背景样本数），实现精度与效率的权衡。

**示例**：
```python
import shap
import tensorflow as tf

# 假设 model 是 Keras 模型
model = tf.keras.models.load_model('my_model.h5')

# 选择背景样本（训练数据子集）
background = X_train[:100]

# 创建解释器
explainer = shap.DeepExplainer(model, background)

# 计算 SHAP 值
shap_values = explainer.shap_values(X_test[:10])
```

### KernelExplainer

**功能**：使用带权线性回归的 Kernel SHAP 方法实现模型无关的 SHAP 值计算。

**构造参数**：
- `model`：接收样本矩阵并返回模型输出的函数或模型对象
- `data`：模拟缺失特征的背景数据集（numpy 数组、pandas DataFrame 或稀疏矩阵）
- `feature_names`：可选特征名列表；DataFrame 列名可用时自动推导
- `link`：特征重要性与模型输出的连接函数
  - `"identity"`：直接关联（默认）
  - `"logit"`：适用于概率输出

**核心方法**：
- `shap_values(X, **kwargs)`：计算样本预测的 SHAP 值
  - `nsamples`：单次预测评估次数（"auto" 或整数值）；值越高方差越低
  - `l1_reg`：特征选择正则化（"num_features(int)", "aic", "bic" 或浮点数）
  - 返回数组每行总和等于模型输出与期望值之差
- `explain_row(row)`：解释单次预测并输出归因值与期望值
- `save(file)` / `load(file)`：解释器对象持久化存储

**适用场景**：
- 无专用解释器的黑盒模型
- 处理自定义预测函数时
- 适用于任意模型类型（神经网络、SVM、集成方法等）
- 需要模型无关解释时
- **注意**：速度慢于专用解释器，仅在没有专用方案时使用

**示例**：
```python
import shap
from sklearn.svm import SVC

# 训练模型
model = SVC(probability=True).fit(X_train, y_train)

# 创建预测函数
predict_fn = lambda x: model.predict_proba(x)[:, 1]

# 选择背景样本
background = shap.sample(X_train, 100)

# 创建解释器
explainer = shap.KernelExplainer(predict_fn, background)

# 计算 SHAP 值（可能较慢）
shap_values = explainer.shap_values(X_test[:10])
```

### LinearExplainer

**功能**：考虑特征相关性的线性模型专用解释器。

**构造参数**：
- `model`：线性模型或 (系数, 截距) 元组
- `masker`：特征相关性背景数据
- `feature_perturbation`：特征相关性处理方式
  - `"interventional"`：假设特征独立
  - `"correlation_dependent"`：考虑特征相关性

**支持模型**：
- scikit-learn 线性模型 (LinearRegression, LogisticRegression, Ridge, Lasso, ElasticNet)
- 含系数和截距的自定义线性模型

**适用场景**：
- 线性回归与逻辑回归模型
- 特征相关性对解释精度至关重要时
- 需要极速解释时
- 适用于 GLM 等线性模型

**示例**：
```python
import shap
from sklearn.linear_model import LogisticRegression

# 训练模型
model = LogisticRegression().fit(X_train, y_train)

# 创建解释器
explainer = shap.LinearExplainer(model, X_train)

# 计算 SHAP 值
shap_values = explainer.shap_values(X_test)
```

### GradientExplainer

**功能**：使用期望梯度为神经网络近似计算 SHAP 值。

**构造参数**：
- `model`：深度学习模型（TensorFlow 或 PyTorch）
- `data`：积分背景样本
- `batch_size`：梯度计算批大小
- `local_smoothing`：平滑噪声添加量（默认 0）

**适用场景**：
- 作为 DeepExplainer 的神经网络替代方案
- 优先选择基于梯度的解释时
- 适用于可获取梯度信息的可微分模型

**示例**：
```python
import shap
import torch

# 假设 model 是 PyTorch 模型
model = torch.load('model.pt')

# 选择背景样本
background = X_train[:100]

# 创建解释器
explainer = shap.GradientExplainer(model, background)

# 计算 SHAP 值
shap_values = explainer.shap_values(X_test[:10])
```

### PermutationExplainer

**功能**：通过输入排列迭代近似计算 Shapley 值。

**构造参数**：
- `model`：预测函数
- `masker`：背景数据或掩码对象
- `max_evals`：单样本最大模型评估次数

**适用场景**：
- 需要精确 Shapley 值但无专用解释器时
- 特征集较小且排列计算可行时
- 作为 KernelExplainer 的高精度替代方案（速度更慢）

**示例**：
```python
import shap

# 创建解释器
explainer = shap.PermutationExplainer(model.predict, X_train)

# 计算 SHAP 值
shap_values = explainer.shap_values(X_test[:10])
```

## 解释器选择指南

**解释器决策树**：

1. **是否为树模型？** (XGBoost, LightGBM, CatBoost, 随机森林等)
   - 是 → 使用 `TreeExplainer`（快速精确）
   - 否 → 进入第 2 步

2. **是否为深度神经网络？** (TensorFlow, PyTorch, Keras)
   - 是 → 使用 `DeepExplainer` 或 `GradientExplainer`
   - 否 → 进入第 3 步

3. **是否为线性模型？** (线性/逻辑回归, GLM)
   - 是 → 使用 `LinearExplainer`（极速）
   - 否 → 进入第 4 步

4. **是否需要模型无关解释？**
   - 是 → 使用 `KernelExplainer`（较慢但通用）
   - 计算资源充足且需高精度 → 使用 `PermutationExplainer`

5. **不确定或需要自动选择？**
   - 使用 `shap.Explainer`（自动选择最优算法）

## 通用参数说明

**背景数据/掩码器**：
- 作用：代表"典型"输入以建立基准期望
- 规模建议：50-1000 样本（复杂模型需更多）
- 选择：训练数据随机采样或 kmeans 代表性样本

**特征名称**：
- 从 pandas DataFrame 自动提取
- 可手动指定 numpy 数组特征名
- 对可视化可解释性至关重要

**模型输出规范**：
- 原始模型输出 vs 转换输出（概率、对数几率）
- 直接影响 SHAP 值的正确解读
- 示例：XGBoost 分类器中，SHAP 解释的是逻辑转换前的边际输出（对数几率）

## 性能考量

**速度排名**（从快到慢）：
1. `LinearExplainer` - 近实时
2. `TreeExplainer` - 极快，扩展性强
3. `DeepExplainer` - 神经网络场景快速
4. `GradientExplainer` - 神经网络场景快速
5. `KernelExplainer` - 较慢，必要时使用
6. `PermutationExplainer` - 极慢但小特征集精度最高

**内存考量**：
- `TreeExplainer`：内存开销低
- `DeepExplainer`：内存占用与背景样本量成正比
- `KernelExplainer`：大型背景数据集可能内存密集
- 大型数据集建议：使用分批处理或子采样

## 解释器输出：Explanation 对象

所有解释器返回的 `shap.Explanation` 对象包含：
- `values`：SHAP 值（numpy 数组）
- `base_values`：期望模型输出（基线）
- `data`：原始特征值
- `feature_names`：特征名称

Explanation 对象支持：
- 切片：`explanation[0]` 获取首个样本
- 数组操作：兼容 numpy 运算
- 直接绘图：可传入绘图函数
