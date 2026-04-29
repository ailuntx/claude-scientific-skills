# 竞争风险分析

## 概述

当研究对象可能经历多种互斥事件（事件类型）之一时，就存在竞争风险。当某一事件发生时，它会阻止（"竞争"）其他事件的发生。

### 竞争风险示例

**医学研究：**
- 癌症死亡 vs. 心血管疾病死亡 vs. 其他原因死亡
- 癌症研究中复发 vs. 未复发死亡
- 移植患者中不同类型的感染

**其他应用：**
- 工作终止：退休 vs. 辞职 vs. 解雇
- 设备故障：不同故障模式
- 客户流失：不同流失原因

### 核心概念：累积发生率函数 (CIF)

**累积发生率函数 (CIF)** 表示在考虑竞争风险存在的情况下，特定事件类型在时间 *t* 前发生的概率。

**CIF_k(t) = P(T ≤ t, 事件类型 = k)**

这与Kaplan-Meier估计量不同，后者在存在竞争风险时会高估事件概率。

## 何时使用竞争风险分析

**适用场景：**
- 存在多种互斥事件类型
- 某一事件的发生会阻止其他事件
- 需要估计特定事件类型的概率
- 需了解协变量如何影响不同事件类型

**不适用场景：**
- 仅关注单一事件类型（使用标准生存分析）
- 事件非互斥（使用重复事件方法）
- 竞争事件极为罕见（可视为删失处理）

## 竞争风险下的累积发生率

### cumulative_incidence_competing_risks 函数

估计各事件类型的累积发生率函数。

```python
from sksurv.nonparametric import cumulative_incidence_competing_risks
from sksurv.datasets import load_leukemia

# 加载含竞争风险的数据
X, y = load_leukemia()
# y包含事件类型：0=删失, 1=复发, 2=死亡

# 计算各事件类型的累积发生率
# 返回：时间点, 事件1的CIF, 事件2的CIF...
time_points, cif_1, cif_2 = cumulative_incidence_competing_risks(y)

# 绘制累积发生率函数
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.step(time_points, cif_1, where='post', label='复发', linewidth=2)
plt.step(time_points, cif_2, where='post', label='缓解期死亡', linewidth=2)
plt.xlabel('时间（周）')
plt.ylabel('累积发生率')
plt.title('竞争风险：复发 vs 死亡')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

### 结果解读

- **时间t的CIF**：在时间t前发生该特定事件的概率
- **所有CIF之和**：发生任何事件的总概率（全因概率）
- **1 - CIF总和**：未发生事件且未被删失的概率

## 竞争风险数据格式

### 创建含事件类型的结构化数组

```python
import numpy as np
from sksurv.util import Surv

# 事件类型：0=删失, 1=事件类型1, 2=事件类型2
event_types = np.array([0, 1, 2, 1, 0, 2, 1])
times = np.array([10.2, 5.3, 8.1, 3.7, 12.5, 6.8, 4.2])

# 创建生存数组
# 竞争风险场景：发生任何事件时event=True
# 单独存储事件类型或在event字段编码
y = Surv.from_arrays(
    event=(event_types > 0),  # 发生事件时为True
    time=times
)

# 保留event_types用于区分事件类型
```

### 转换含事件类型的数据

```python
import pandas as pd
from sksurv.util import Surv

# 假设数据含列：time, event_type
# event_type: 0=删失, 1=类型1, 2=类型2...

df = pd.read_csv('competing_risks_data.csv')

# 创建生存结果
y = Surv.from_arrays(
    event=(df['event_type'] > 0),
    time=df['time']
)

# 存储事件类型
event_types = df['event_type'].values
```

## 组间累积发生率比较

### 分层分析

```python
from sksurv.nonparametric import cumulative_incidence_competing_risks
import matplotlib.pyplot as plt

# 按治疗组分组
mask_treatment = X['treatment'] == 'A'
mask_control = X['treatment'] == 'B'

y_treatment = y[mask_treatment]
y_control = y[mask_control]

# 计算各组的CIF
time_trt, cif1_trt, cif2_trt = cumulative_incidence_competing_risks(y_treatment)
time_ctl, cif1_ctl, cif2_ctl = cumulative_incidence_competing_risks(y_control)

# 绘制比较图
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

# 事件类型1
ax1.step(time_trt, cif1_trt, where='post', label='治疗组', linewidth=2)
ax1.step(time_ctl, cif1_ctl, where='post', label='对照组', linewidth=2)
ax1.set_xlabel('时间')
ax1.set_ylabel('累积发生率')
ax1.set_title('事件类型1')
ax1.legend()
ax1.grid(True, alpha=0.3)

# 事件类型2
ax2.step(time_trt, cif2_trt, where='post', label='治疗组', linewidth=2)
ax2.step(time_ctl, cif2_ctl, where='post', label='对照组', linewidth=2)
ax2.set_xlabel('时间')
ax2.set_ylabel('累积发生率')
ax2.set_title('事件类型2')
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

## 竞争风险统计检验

### Gray检验

使用Gray检验比较组间累积发生率函数（可通过lifelines等包实现）。

```python
# 注意：scikit-survival未直接提供Gray检验
# 建议使用lifelines或其他包

# from lifelines.statistics import multivariate_logrank_test
# result = multivariate_logrank_test(times, groups, events, event_of_interest=1)
```

## 竞争风险建模

### 方法1：事件特异性风险模型

为每个事件类型单独拟合Cox模型，将其他事件类型视为删失。

```python
from sksurv.linear_model import CoxPHSurvivalAnalysis
from sksurv.util import Surv

# 为各事件类型创建独立结果
# 事件类型1：将类型2视为删失
y_event1 = Surv.from_arrays(
    event=(event_types == 1),
    time=times
)

# 事件类型2：将类型1视为删失
y_event2 = Surv.from_arrays(
    event=(event_types == 2),
    time=times
)

# 拟合事件特异性模型
cox_event1 = CoxPHSurvivalAnalysis()
cox_event1.fit(X, y_event1)

cox_event2 = CoxPHSurvivalAnalysis()
cox_event2.fit(X, y_event2)

# 解读各事件类型的系数
print("事件类型1（如复发）：")
print(cox_event1.coef_)

print("\n事件类型2（如死亡）：")
print(cox_event2.coef_)
```

**结果解读：**
- 为每个竞争事件建立独立模型
- 系数反映对特定事件类型风险率的影响
- 某协变量可能增加一类事件风险但降低另一类风险

### 方法2：Fine-Gray亚分布风险模型

直接建模累积发生率（scikit-survival未直接提供，需借助其他包）。

```python
# 注意：scikit-survival未直接提供Fine-Gray模型
# 建议使用lifelines或rpy2调用R的cmprsk包

# from lifelines import CRCSplineFitter
# crc = CRCSplineFitter()
# crc.fit(df, event_col='event', duration_col='time')
```

## 完整竞争风险分析实例

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sksurv.nonparametric import cumulative_incidence_competing_risks
from sksurv.linear_model import CoxPHSurvivalAnalysis
from sksurv.util import Surv

# 模拟竞争风险数据
np.random.seed(42)
n = 200

# 创建特征
age = np.random.normal(60, 10, n)
treatment = np.random.choice(['A', 'B'], n)

# 模拟事件时间和类型
# 事件类型：0=删失, 1=复发, 2=死亡
times = np.random.exponential(100, n)
event_types = np.zeros(n, dtype=int)

# 高龄增加两类事件风险，治疗A降低复发风险
for i in range(n):
    if times[i] < 150:  # 发生事件
        # 各事件类型概率
        p_relapse = 0.6 if treatment[i] == 'B' else 0.4
        event_types[i] = 1 if np.random.rand() < p_relapse else 2
    else:
        times[i] = 150  # 研究结束时删失

# 创建DataFrame
df = pd.DataFrame({
    'time': times,
    'event_type': event_types,
    'age': age,
    'treatment': treatment
})

# 编码治疗类型
df['treatment_A'] = (df['treatment'] == 'A').astype(int)

# 1. 总体累积发生率
print("=" * 60)
print("总体累积发生率")
print("=" * 60)

y_all = Surv.from_arrays(event=(df['event_type'] > 0), time=df['time'])
time_points, cif_relapse, cif_death = cumulative_incidence_competing_risks(y_all)

plt.figure(figsize=(10, 6))
plt.step(time_points, cif_relapse, where='post', label='复发', linewidth=2)
plt.step(time_points, cif_death, where='post', label='死亡', linewidth=2)
plt.xlabel('时间（天）')
plt.ylabel('累积发生率')
plt.title('竞争风险：复发 vs 死亡')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()

print(f"5年复发率: {cif_relapse[-1]:.2%}")
print(f"5年死亡率: {cif_death[-1]:.2%}")

# 2. 按治疗分层
print("\n" + "=" * 60)
print("按治疗分层的累积发生率")
print("=" * 60)

for trt in ['A', 'B']:
    mask = df['treatment'] == trt
    y_trt = Surv.from_arrays(
        event=(df.loc[mask, 'event_type'] > 0),
        time=df.loc[mask, 'time']
    )
    time_trt, cif1_trt, cif2_trt = cumulative_incidence_competing_risks(y_trt)
    print(f"\n治疗组{trt}:")
    print(f"  5年复发率: {cif1_trt[-1]:.2%}")
    print(f"  5年死亡率: {cif2_trt[-1]:.2%}")

# 3. 事件特异性模型
print("\n" + "=" * 60)
print("事件特异性风险模型")
print("=" * 60)

X = df[['age', 'treatment_A']]

# 复发模型（事件类型1）
y_relapse = Surv.from_arrays(
    event=(df['event_type'] == 1),
    time=df['time']
)
cox_relapse = CoxPHSurvivalAnalysis()
cox_relapse.fit(X, y_relapse)

print("\n复发模型:")
print(f"  年龄:        HR = {np.exp(cox_relapse.coef_[0]):.3f}")
print(f"  治疗A:       HR = {np.exp(cox_relapse.coef_[1]):.3f}")

# 死亡模型（事件类型2）
y_death = Surv.from_arrays(
    event=(df['event_type'] == 2),
    time=df['time']
)
cox_death = CoxPHSurvivalAnalysis()
cox_death.fit(X, y_death)

print("\n死亡模型:")
print(f"  年龄:        HR = {np.exp(cox_death.coef_[0]):.3f}")
print(f"  治疗A:       HR = {np.exp(cox_death.coef_[1]):.3f}")

print("\n" + "=" * 60)
```

## 重要注意事项

### 竞争风险中的删失

- **管理性删失**：研究结束时对象仍处于风险中
- **失访**：对象在事件发生前退出研究
- **竞争事件**：其他事件发生 - 对CIF不是删失，但对事件特异性模型需视为删失

### 事件特异性模型与亚分布模型选择

**事件特异性风险模型：**
- 更易解释
- 直接反映对风险率的影响
- 更适合病因学研究
- 可通过scikit-survival实现

**Fine-Gray亚分布模型：**
- 直接建模累积发生率
- 更适合预测和风险分层
- 更适用于临床决策
- 需借助其他包实现

### 常见错误

**错误1**：使用Kaplan-Meier估计事件特异性概率
- **错误做法**：将类型2视为删失，用Kaplan-Meier估计类型1
- **正确做法**：使用考虑竞争风险的累积发生率函数

**错误2**：在竞争风险显著时忽略其存在
- 当竞争事件率 > 10-20% 时，必须使用竞争风险方法

**错误3**：混淆事件特异性和亚分布风险
- 二者回答不同问题
- 根据研究问题选择合适模型

## 总结

**核心函数：**
- `cumulative_incidence_competing_risks`：估计各事件类型的CIF
- 为事件特异性风险拟合独立Cox模型
- 使用分层分析进行组间比较

**最佳实践：**
1. 始终绘制累积发生率函数
2. 同时报告事件特异性和总体发生率
3. 在scikit-survival中使用事件特异性模型
4. 预测时考虑Fine-Gray模型（其他包）
5. 明确区分竞争事件与删失事件
