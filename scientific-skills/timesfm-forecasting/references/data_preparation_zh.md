# TimesFM 的数据准备

## 输入格式

TimesFM 接受一个**一维 numpy 数组列表**。每个数组代表一个单变量时间序列。

```python
inputs = [
    np.array([1.0, 2.0, 3.0, 4.0, 5.0]),       # 序列1
    np.array([10.0, 20.0, 15.0, 25.0]),          # 序列2 (不同长度)
    np.array([100.0, 110.0, 105.0, 115.0, 120.0, 130.0]),  # 序列3
]
```

### 关键特性

- **可变长度**：同一批次中的序列可具有不同长度
- **浮点数值**：使用 `np.float32` 或 `np.float64`
- **仅限一维**：每个数组必须是一维的（非二维矩阵行）
- **NaN 处理**：开头的 NaN 会被去除；中间的 NaN 会进行线性插值

## 从常见格式加载

### CSV — 单序列（长格式）

```python
import pandas as pd
import numpy as np

df = pd.read_csv("data.csv", parse_dates=["date"])
values = df["value"].values.astype(np.float32)
inputs = [values]
```

### CSV — 多序列（宽格式）

```python
df = pd.read_csv("data.csv", parse_dates=["date"], index_col="date")
inputs = [df[col].dropna().values.astype(np.float32) for col in df.columns]
```

### CSV — 带 ID 列的长格式

```python
df = pd.read_csv("data.csv", parse_dates=["date"])
inputs = []
for series_id, group in df.groupby("series_id"):
    values = group.sort_values("date")["value"].values.astype(np.float32)
    inputs.append(values)
```

### Pandas DataFrame

```python
# 单列
inputs = [df["temperature"].values.astype(np.float32)]

# 多列
inputs = [df[col].dropna().values.astype(np.float32) for col in numeric_cols]
```

### Numpy 数组

```python
# 二维数组 (行=序列, 列=时间步)
data = np.load("timeseries.npy")  # 形状 (N, T)
inputs = [data[i] for i in range(data.shape[0])]

# 或从一维数组创建
inputs = [np.sin(np.linspace(0, 10, 200))]
```

### Excel

```python
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
inputs = [df[col].dropna().values.astype(np.float32) for col in df.select_dtypes(include=[np.number]).columns]
```

### Parquet

```python
df = pd.read_parquet("data.parquet")
inputs = [df[col].dropna().values.astype(np.float32) for col in df.select_dtypes(include=[np.number]).columns]
```

### JSON

```python
import json

with open("data.json") as f:
    data = json.load(f)

# 假设格式为 {"series_name": [values...], ...}
inputs = [np.array(values, dtype=np.float32) for values in data.values()]
```

## NaN 处理

TimesFM 自动处理 NaN 值：

### 开头的 NaN

输入模型前会被去除：

```python
# 输入:  [NaN, NaN, 1.0, 2.0, 3.0]
# 实际: [1.0, 2.0, 3.0]
```

### 中间的 NaN

进行线性插值：

```python
# 输入:  [1.0, NaN, 3.0, NaN, NaN, 6.0]
# 实际: [1.0, 2.0, 3.0, 4.0, 5.0, 6.0]
```

### 结尾的 NaN

**不处理** — 需在传入模型前去除：

```python
values = df["value"].values.astype(np.float32)
# 去除结尾的 NaN
while len(values) > 0 and np.isnan(values[-1]):
    values = values[:-1]
inputs = [values]
```

### 最佳实践

```python
def clean_series(arr: np.ndarray) -> np.ndarray:
    """为 TimesFM 输入清理时间序列"""
    arr = np.asarray(arr, dtype=np.float32)
    # 去除结尾的 NaN
    while len(arr) > 0 and np.isnan(arr[-1]):
        arr = arr[:-1]
    # 将 inf 替换为 NaN (后续会插值)
    arr[np.isinf(arr)] = np.nan
    return arr

inputs = [clean_series(df[col].values) for col in cols]
```

## 上下文长度考量

| 上下文长度 | 使用场景 | 说明 |
| -------------- | -------- | ----- |
| 64–256 | 快速原型设计 | 最小上下文，速度快 |
| 256–512 | 日度数据，约1年 | 良好平衡 |
| 512–1024 | 日度数据，约2-3年 | 标准生产环境 |
| 1024–4096 | 小时数据，周模式 | 上下文越长效果越好 |
| 4096–16384 | 高频数据，长模式 | TimesFM 2.5 最大值 |

**经验法则**：至少提供主导模式的3-5个完整周期
（例如：日度数据的周季节性，至少提供21-35天）。

## 协变量 (XReg)

TimesFM 2.5 通过 `forecast_with_covariates()` API 支持外生变量。

### 协变量类型

| 类型 | 描述 | 示例 |
| ---- | ----------- | ------- |
| **动态数值型** | 随时间变化的数值特征 | 温度、价格、促销支出 |
| **动态类别型** | 随时间变化的类别特征 | 星期几、节假日标志 |
| **静态类别型** | 每个序列的固定特征 | 店铺ID、区域、产品类别 |

### 准备协变量

每个协变量对每个序列必须具有 `context + horizon` 的长度：

```python
import numpy as np

context_len = 100   # 历史数据长度
horizon = 24        # 预测范围
total_len = context_len + horizon

# 动态数值型：每个序列的温度预测
temp = [
    np.random.randn(total_len).astype(np.float32),  # 序列1
    np.random.randn(total_len).astype(np.float32),  # 序列2
]

# 动态类别型：每个序列的星期几 (0-6)
dow = [
    np.tile(np.arange(7), total_len // 7 + 1)[:total_len],  # 序列1
    np.tile(np.arange(7), total_len // 7 + 1)[:total_len],  # 序列2
]

# 静态类别型：每个序列一个标签
regions = ["east", "west"]

# 使用协变量预测
point, quantiles = model.forecast_with_covariates(
    inputs=[values1, values2],
    dynamic_numerical_covariates={"temperature": temp},
    dynamic_categorical_covariates={"day_of_week": dow},
    static_categorical_covariates={"region": regions},
    xreg_mode="xreg + timesfm",
)
```

### XReg 模式

| 模式 | 描述 |
| ---- | ----------- |
| `"xreg + timesfm"` | 先处理协变量，再与 TimesFM 预测结果组合 |
| `"timesfm + xreg"` | 先进行 TimesFM 预测，再用协变量调整 |

## 常见数据问题

### 问题：序列过短

TimesFM 至少需要1个数据点，但上下文越长预测效果越好。

```python
MIN_LENGTH = 32  # 有意义的预测所需最小长度

inputs = [
    arr for arr in raw_inputs
    if len(arr[~np.isnan(arr)]) >= MIN_LENGTH
]
```

### 问题：恒定值序列

恒定序列可能产生 NaN 或零宽度的预测区间：

```python
for i, arr in enumerate(inputs):
    if np.std(arr[~np.isnan(arr)]) < 1e-10:
        print(f"⚠️ 序列 {i} 为恒定值 — 预测结果将为平线")
```

### 问题：极端异常值

即使经过归一化，大异常值仍可能破坏预测稳定性：

```python
def clip_outliers(arr: np.ndarray, n_sigma: float = 5.0) -> np.ndarray:
    """裁剪超出 n_sigma 标准差的数值"""
    mu = np.nanmean(arr)
    sigma = np.nanstd(arr)
    if sigma > 0:
        arr = np.clip(arr, mu - n_sigma * sigma, mu + n_sigma * sigma)
    return arr
```

### 问题：批次中混合频率

TimesFM 独立处理每个序列，因此可混合不同频率：

```python
inputs = [
    daily_sales,      # 365个点
    weekly_revenue,   # 52个点
    monthly_users,    # 24个点
]
# 所有序列在同一批次预测 — TimesFM 处理不同长度
point, q = model.forecast(horizon=12, inputs=inputs)
```

但 `horizon` 是共享的。若需不同序列不同预测范围，需分开调用预测函数。
